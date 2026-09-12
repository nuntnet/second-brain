---
name: feedback_commit_push_along_the_way
description: "ผู้ใช้สั่ง 'commit push ระหว่างทางเลย' — ให้ commit+push ทีละชิ้นที่ตรวจเสร็จ ไม่ต้องรอจบงานทั้งก้อน และเลื่อน submodule ref ให้ครบทุกชั้น"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: fa22ebb1-f715-4667-85a1-61ebbcc816ab
  modified: 2026-09-12T23:18:46.835Z
---

**สั่งเมื่อ 2026-09-13 ระหว่างรัน agent หลายตัวขนานกัน:** "commit push ระหว่างทางเลย"

**Why:** งานที่วิ่งขนานหลายสายแล้วกองรอ push ตอนจบ ทำให้มองไม่เห็นความคืบหน้าและเสี่ยงชนกับ
Codex ที่ทำงานบน branch เดียวกัน ([[feedback_user_runs_codex_in_parallel]]) · การ push ทีละชิ้น
ยังทำให้ CI เริ่มวิ่งเร็วขึ้นด้วย

**How to apply:** ทุกครั้งที่ตรวจงานชิ้นหนึ่งจบ (gate เขียว + ดู diff แล้วว่าไม่กินไฟล์คนอื่น)
→ commit + push ทันที ไม่ต้องรวบรอ ไม่ต้องถามซ้ำ ครอบทั้งสามชั้น:

1. commit ใน submodule
2. push submodule
3. `git add <submodule>` ที่โมโนรีโป + commit + push ref

ข้อควรระวังที่ใช้จริงในรอบนั้น:
- **stage เฉพาะ path ที่ตัวเองแตะ** ห้าม `git add .` / `-A` เพราะในทรีมีงานค้างของ agent อื่น
  และของ Codex (เช่น `useTheme.ts`) ปนอยู่ตลอด
- ตรวจ `git show --stat <sha>` ก่อน push ว่าไม่มีไฟล์ของคนอื่นติดไป
- **submodule แต่ละตัวอาจอยู่คนละ branch** — รอบนั้น 3rdparty-api อยู่
  `codex/oc-4344-web-member-identity` ขณะที่ตัวอื่นอยู่ `codex/oc-4344-customer-app-integration`
  push ผิด branch = งานหาย

เชื่อม [[feedback_parallel_sessions_git_safety]] [[feedback_decisive_deep_execution]]
