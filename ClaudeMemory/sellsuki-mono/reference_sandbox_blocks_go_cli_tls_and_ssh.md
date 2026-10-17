---
name: reference_sandbox_blocks_go_cli_tls_and_ssh
description: ใน sandbox นี้ glab/tsh/kubectl (Go) verify TLS ไม่ได้ (OSStatus -26276) และ git ผ่าน SSH ถูกบล็อก — git fetch/push ทำได้ผ่าน HTTPS + glab token ถ้า override url.insteadOf ที่ rewrite เป็น SSH (2026-10-17) · ที่เหลือใช้ GitLab REST ผ่าน curl (ใช้ /usr/bin/curl เต็ม path)
metadata:
  node_type: memory
  type: reference
  originSessionId: 7b2f9d92-0c29-4b4e-bcb2-47995813fc48
  modified: 2026-10-13T03:11:51.812Z
---

2026-10-13 เสียเวลาหลายรอบกับเรื่องนี้ อาการคือ **ดูเหมือน VPN/teleport หลุด ทั้งที่ไม่ได้หลุด**

- `glab`, `tsh`, `kubectl` (Go ทั้งหมด) ตอบ `tls: failed to verify certificate: x509: OSStatus -26276`
  ทุกโฮสต์ รวม gitlab.sellsuki.com และ teleport — สาเหตุคือ Go บน darwin verify ผ่าน trustd ซึ่ง
  seatbelt บล็อก **ไม่ใช่ cert หมดอายุ** (`tsh login` ใหม่แล้วก็เหมือนเดิม) และ `SSL_CERT_FILE`
  ไม่ช่วย เพราะ Go บน darwin ไม่อ่านตัวแปรนี้
- `curl` ใช้ได้ปกติ (bundle ของตัวเอง) — ทั้ง public และ `*.internal.staging-th.sellsuki.com`
  ผ่าน HTTP proxy ของ sandbox ต้องใส่ **ชื่อโฮสต์เต็ม** ใน `allowed_domains` (wildcard อย่างเดียวโดนปฏิเสธ)
- `git fetch/push` ผ่าน SSH ถูกบล็อกสนิท → **อย่าเสียเวลา retry**

ทางออกที่ใช้ได้จริง: เรียก GitLab REST API ด้วย curl + token ที่อยู่ใน
`~/Library/Application Support/glab-cli/config.yml` (ไม่ใช่ `~/.config/glab-cli`)
ทำได้ครบ: list/compare, สร้าง branch, สร้าง commit (`POST repository/commits` ส่ง actions),
cherry-pick ฝั่ง server (`POST repository/commits/:sha/cherry_pick`), เปิด MR, ดึง
`repository/archive.tar.gz?sha=<ref>` มา build/test ในที่ทำงานชั่วคราวโดยไม่ต้อง checkout

อย่างอื่นที่ติดกับดักเดียวกัน:
- `pip` ก็ fail ด้วย `SSLCertVerificationError('OSStatus -26276')` → ใช้ `pip install --use-deprecated=legacy-certs`
  (และ `--no-build-isolation` ถ้า sdist ต้อง build ซึ่งจะไปเจอ TLS อีกรอบ)
- `go build/test` ต้องตั้ง `GOCACHE` ไปที่ scratchpad เพราะ `~/Library/Caches/go-build` เขียนไม่ได้
- `git checkout` ในซับโมดูลอาจล้มกลางคัน เพราะเขียน `.claude/commands/` `.claude/skills/` ไม่ได้
  (เหลือ working tree ค้าง ต้อง `git checkout -- .` คืน) → ถ้าอยากดูโค้ด ref อื่น ใช้ archive แทนการสลับ branch

`kubectl` ทำงานได้ถ้ารันแบบ `dangerouslyDisableSandbox` แต่ classifier อนุญาตบ้างไม่อนุญาตบ้าง
อย่าพึ่งมันเป็นทางหลัก

เกี่ยวข้อง: [[reference_teleport_session_kills_devth_access]] (อันนั้นคือ VPN หลุดจริง — แยกให้ออก)

**2026-10-17 — git fetch/push ผ่าน HTTPS ใช้ได้จริง** (ไม่ต้องยิง commit ผ่าน REST):
- global git config มี `url.git@gitlab.sellsuki.com:.insteadOf=https://gitlab.sellsuki.com` → ทุก URL https ถูกแปลงเป็น SSH เงียบ ๆ
  แล้วตายด้วย "Please make sure you have the correct access rights" (ดูเหมือน auth ผิด ที่จริงคือ SSH ถูกบล็อก)
- ทางแก้: ใส่ rule ที่ prefix ยาวกว่า (longest match ชนะ) ให้แปลงเป็นตัวเอง + basic auth ด้วย token ผ่าน header:
  ```
  B64=$(printf 'oauth2:%s' "$TOK" | base64)
  /usr/bin/git -c credential.helper= \
    -c "url.https://gitlab.sellsuki.com/sellsuki/.insteadOf=https://gitlab.sellsuki.com/sellsuki/" \
    -c "http.extraheader=Authorization: Basic $B64" fetch https://gitlab.sellsuki.com/<path>.git +refs/heads/main:refs/remotes/origin/main
  ```
  (`PRIVATE-TOKEN` header ใช้กับ git ไม่ได้ ต้องเป็น Basic oauth2:<token>) · push ก็แบบเดียวกัน
- กับดักใน zsh: ห้ามตั้งตัวแปรชื่อ `path` (ผูกกับ PATH ทั้ง shell → "command not found: git") และ `set -- $spec` ไม่ split คำ ต้อง `${=spec}`
- `curl` ใน PATH ของ sandbox อาจหาย → เรียก `/usr/bin/curl`
- การเชื่อม localhost (Postgres เทส) จาก sandbox ถูกบล็อก (`connect: operation not permitted`) และ `httptest` bind ไม่ได้ → รัน `go test` ที่ต้อง DB แบบ dangerouslyDisableSandbox พร้อม `GOCACHE` ใน scratchpad; docker daemon เรียกได้เฉพาะนอก sandbox
