---
name: reference_moving_a_branch_under_a_running_dev_server
description: "ย้าย branch/ลบ worktree ใต้ dev server ที่รันอยู่ → vite เสิร์ฟ .ts ดิบ (\"Unexpected token 'export'\") หรือ 404 · เช็ค cwd ของทุก process ที่ฟัง port ก่อน แล้ว restart ทันที"
metadata: 
  node_type: memory
  type: reference
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
  modified: 2026-09-14T14:31:55.969Z
---

2026-09-14 ระหว่างเคลียร์ branch ([[reference_oc2plus_branch_sprawl_is_worktree_debris]])
สั่ง `git checkout develop && git pull` ขยับ 132 commit ใต้ vite ที่รันมาตั้งแต่
3 วันก่อน ผลคือ **vite เสิร์ฟไฟล์ `.ts` ดิบโดยไม่ผ่าน esbuild**

อาการที่เห็นในเบราว์เซอร์: `Uncaught SyntaxError: Unexpected token 'export'
(at readiness.ts:3:1)` — ชี้ไปที่ไฟล์ที่เพิ่งเข้ามากับ branch ใหม่ ทำให้หลงคิดว่า
ไฟล์นั้นผิด · **ไฟล์ไม่ผิด server ต่างหากที่ค้าง**

**ยืนยันให้แน่ก่อนแก้:**
```
curl -s http://localhost:<port>/src/<path>.ts | head -3
```
ถ้าเห็น `export type …` หรือ type annotation กลับมาดิบ ๆ = transform ตาย
(ของดีต้องได้ JS ที่ strip type แล้วและ import ถูก rewrite เป็น `/node_modules/.vite/deps/…`)

**เช็คก่อนย้าย branch / ลบ worktree เสมอ** — หา process ที่ถือ cwd ในโฟลเดอร์นั้น:
```
for p in $(lsof -nP -iTCP -sTCP:LISTEN | awk 'NR>1{print $2}' | sort -u); do
  lsof -a -p $p -d cwd -Fn | grep '^n'
done
```

**ผลของการลบโฟลเดอร์ใต้ server ที่รันอยู่ ต่างกันตามภาษา:**
- **vite/node** → ตอบ **404** ทันที (ถือ cwd ที่ถูกลบ) ต้องฆ่าทิ้ง
- **Go (air)** → binary ยัง **200** ต่อไปเพราะรันจากหน่วยความจำ แต่ air rebuild
  ไม่ได้อีกแล้ว = zombie ที่ดูปกติ · อย่าเพิ่งฆ่าถ้ามีคนใช้อยู่ แต่ต้องบอกเจ้าของ

**restart ยังไงเมื่ออยู่ใต้ overmind** — repo นี้มี socket หลายอัน ไม่ใช่ `.overmind.sock`
อันเดียว (`.overmind-oc2plus-fe.sock`, `.overmind-oc2api.sock`, `.overmind-oc4344.sock`, …)
หา socket ที่ถูกจาก **เวลาไฟล์ socket ตรงกับเวลาที่ process เริ่ม** แล้ว
`OVERMIND_SOCKET=<sock> overmind restart <name>` (ดู [[project_overmind_restart_quirk]] —
`overmind start <name> -D` ใช้ไม่ได้เมื่อ session รันอยู่แล้ว)

⚠️ **เกณฑ์ "merged + clean" ไม่พอสำหรับการลบ worktree** — worktree ที่ merge แล้วและ
สะอาด ยังอาจมี session อื่นรัน server อยู่จากตรงนั้น การลบทำให้เขาพังกลางทาง
เพิ่มเงื่อนไข "ไม่มี process ถือ cwd อยู่" ด้วยเสมอ
