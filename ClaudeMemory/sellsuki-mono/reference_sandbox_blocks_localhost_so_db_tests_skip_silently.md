---
name: reference_sandbox_blocks_localhost_so_db_tests_skip_silently
description: "2026-10-25 sandbox ของ Claude Code บล็อก localhost TCP → เทส Go ที่ใช้ Postgres (internal/testdb) skip เงียบๆ และ httptest.NewServer ล้ม 'bind: operation not permitted' · exit 0 ไม่พิสูจน์ว่า DB test รัน · agent รายงาน 'ไม่มี Postgres' ผิด · ต้องรัน go test -v นับ PASS/SKIP โดย dangerouslyDisableSandbox เฉพาะคำสั่งนั้น (ผ่านด่านอนุญาต) · TMPDIR ของ shell ที่ปิด sandbox ต่างจากปกติ"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
---

chat-core ใช้ `internal/testdb` (default `postgres://postgres:postgres@localhost:5432/chat_core_test`, schema แยกต่อ package แล้ว drop) และ skip ตัวเองเมื่อต่อไม่ได้ → ใน sandbox: เทส repository ถูกข้ามเงียบๆ, เทสที่ `httptest.NewServer` (22 package ใน chat-core, 5 ใน messaging-backend, 8 ใน agent) ล้มเพราะ bind ไม่ได้ · เหตุคือ sandbox ไม่ใช่ไม่มี DB (Postgres ของ stack local รันอยู่ที่ 5432).

**How to apply:** รัน `go test -count=1 -p 1 -v ./...` ด้วย `dangerouslyDisableSandbox: true` เฉพาะคำสั่งเทส (ผ่านด่านอนุญาตปกติ) แล้วนับบรรทัด `--- PASS/SKIP/FAIL` — อย่าเชื่อ exit code อย่างเดียว · ต้องใส่ `timeout` ยาว (~1500 วิ ต่อ chat-core ทั้ง repo, verbose ช้ากว่าปกติ) หรือรันใน background · เขียนผลลง `$TMPDIR` ของ shell ที่ปิด sandbox (`/var/folders/.../T/`) ซึ่งต่างจาก `$TMPDIR` ปกติ (`/tmp/claude-501`) · build chat-core กับ kit ที่ยังไม่ tag ใช้ `-modfile=$TMPDIR/x.mod` ที่ copy go.mod + `replace` ชี้ worktree ของ kit (path module คือ `gitlab.sellsuki.com/sellsuki/share/library/ai-platform-kit-go`) ห้าม commit replace/go.work · agent ของ frontend: `pnpm install` ใน worktree ล้ม (EPERM `.idea`) → symlink node_modules จาก checkout หลัก · worktree ของ repo ที่ track `.claude/commands`/`.idea`: `git worktree add --no-checkout` + `sparse-checkout set '/*' '!/.claude/commands/' '!/.idea/'` · ห้ามให้ agent ลบด้วย `rm -rf $VAR/...` (ด่านบล็อก) ใช้ `git worktree remove --force` ด้วย path เต็ม.
