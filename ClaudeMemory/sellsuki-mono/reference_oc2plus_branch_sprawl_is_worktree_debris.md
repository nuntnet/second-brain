---
name: reference_oc2plus_branch_sprawl_is_worktree_debris
description: "OC2Plus \"โค้ดกระจายหลาย branch\" ที่เห็นคือ worktree/branch ค้างจาก session เก่า ไม่ใช่งานที่ยังไม่ merge — เช็คด้วย rev-list origin/develop..branch ก่อนสรุป"
metadata: 
  node_type: memory
  type: reference
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
  modified: 2026-09-14T14:16:18.790Z
---

2026-09-14 เคลียร์ครั้งใหญ่ 5 repo ของ OC2Plus (backoffice FE/API, member FE/API,
3rdparty-api): **226 branch → 49 · 99 worktree → 24** โดยไม่ต้องรวมงานอะไรเลย

**สิ่งที่ดูเหมือนปัญหา แต่ไม่ใช่** — branch เยอะไม่ได้แปลว่างานกระจาย ตรวจด้วย
`git rev-list --count origin/develop..<branch>` แล้วมีแค่ ~25 จาก 226 ที่มี commit
ไม่อยู่บน develop และเกือบทั้งหมดเป็นของซ้ำ:
- branch "รวมร่างไว้ดู local" (`local-preview/member-screens`, `local/qa-integration`)
  — เนื้องานอยู่บน develop แล้วด้วย commit hash อื่น
- `feat/oc-4506-remove-vue` 13 commit แต่ develop ไม่มี `.vue` เหลือแล้ว
- branch ตัดจาก `main` เก่าก่อน develop แยกสาย (190-255 commit) ตายไปนานแล้ว

⚠️ **`git cherry -v` หลอกได้** — มันเทียบ patch-id ถ้า MR ถูก rebase/squash จะขึ้น `+`
(ยังไม่ merge) ทั้งที่เนื้องานอยู่บน develop แล้ว · **เช็คด้วยเนื้อหาจริง**
(`git ls-tree -r origin/develop | grep <feature>`) ถึงจะเชื่อได้

**สาเหตุจริงที่ทำให้รู้สึกกระจาย: checkout หลักไม่ได้อยู่บน develop** — 3/5 repo
ค้างอยู่บน branch ของ session เก่า (`codex/…`, `local-preview/…`) และอีก 2 repo
behind develop 5-9 commit · แก้ด้วย `git checkout develop && git pull --ff-only`
ทั้ง 5 repo จบ

**สูตรเคลียร์ที่ปลอดภัย**
1. `git worktree prune` ทุก repo
2. ลบ worktree ใต้ `.worktrees/` ที่ `rev-list origin/develop..branch` = 0 **และ**
   `status --porcelain` ว่าง · **ห้ามแตะ `/private/tmp/claude-*/…`** — เป็นของ
   session อื่นที่อาจยังรันอยู่
3. `git branch -d` บน `--merged origin/develop` — `-d` ปฏิเสธเองถ้ายังไม่ merge
   หรือถูก checkout ใน worktree อื่น จึงปลอดภัยโดยโครงสร้าง

🔥 **บทเรียนที่เจ็บ: ลบ worktree = ฆ่า dev server ที่รันจากตรงนั้น** — 5192 กับ 5194
กลายเป็น 404 ทันที (vite ถือ cwd ที่ถูกลบ) โค้ดไม่หายเพราะ merge เข้า develop
หมดแล้ว แต่ต้องปิด process ทิ้งแล้วรันใหม่จาก checkout หลัก · **เช็ค
`lsof -a -p <pid> -d cwd` ก่อนลบ worktree ถ้ามี server รันอยู่**

ดู [[reference_worktree_remove_rewinds_main_checkout]] และ
[[feedback_parallel_sessions_git_safety]]
