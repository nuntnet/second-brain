---
name: reference_glab_auto_merge_does_not_wait
description: glab mr merge --auto-merge ไม่ได้รอ pipeline บน GitLab นี้ — มัน merge ทันทีทั้งที่ pipeline ยังรัน ถ้าสั่งว่า merge ตอน CI เขียว ต้องรอ pipeline เองก่อนแล้วค่อย merge
metadata: 
  node_type: memory
  type: reference
  originSessionId: d0b39379-feaa-4fd5-b115-18617c202159
  modified: 2026-09-14T02:37:37.619Z
---

**`glab mr merge --auto-merge` ไม่รับประกันว่าจะรอ pipeline** (ยืนยัน 2026-09-14 บน `bola-backend` MR !183)

ผมตั้งใจใช้ flag นี้แทน "merge เมื่อ CI เขียว" ตามที่ผู้ใช้สั่ง ผลคือมัน **merge ทันที**

```
$ glab mr merge 183 -R ... --auto-merge --yes
! Pipeline status: running
✓ Merged!
```

ยืนยันจาก API: `merged_at = 01:46:07` ขณะที่ pipeline 58661 ยัง `status=running` และ `finished_at=None`

`glab mr merge --help` เขียนแค่ `--auto-merge  Set auto-merge. (true)` ซึ่งอ่านแล้วเข้าใจได้ว่าเป็น merge-when-pipeline-succeeds แต่พฤติกรรมจริงบน instance นี้ไม่ใช่ (น่าจะเพราะ project ไม่ได้เปิด MWPS มันเลย fall through ไป merge เลย)

**ผลรอบนี้:** โชคดีที่ pipeline จบแล้วเขียว และ commit ชุดเดียวกันผ่าน CI บน develop + deploy ขึ้น dev มาก่อนแล้ว ความเสียหายจึงไม่เกิด แต่นั่นคือโชค ไม่ใช่การควบคุม

**How to apply:** ถ้าคำสั่งคือ "merge ตอน CI เขียว" ให้ **รอ pipeline ด้วยตัวเองก่อน** (Monitor หรือ poll `pipelines/<id>` จน `status=success`) แล้วค่อยสั่ง `glab mr merge` ธรรมดา อย่าพึ่ง `--auto-merge` และอย่ารายงานว่า "merge ตอนเขียวแล้ว" ถ้าไม่ได้เห็น `status=success` ด้วยตาตัวเอง

เกี่ยวกับ [[reference_manual_staging_gate_silent_drift]] [[reference_glab_ci_status_stale_pipeline]] [[project_oc4523_line_login_cards]]
