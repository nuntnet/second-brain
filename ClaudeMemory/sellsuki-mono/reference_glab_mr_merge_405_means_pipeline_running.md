---
name: reference_glab_mr_merge_405_means_pipeline_running
description: glab mr merge ตอบ 405 Method Not Allowed = pipeline ยังไม่จบ ไม่ใช่ไม่มีสิทธิ์ — รอ pipeline ให้ success แล้วสั่งใหม่ได้เลย
metadata: 
  node_type: memory
  type: reference
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
  modified: 2026-09-13T15:25:19.882Z
---

`rtk proxy glab mr merge <n> -R <repo> --yes` ตอบ:

```
#1: PUT https://gitlab.sellsuki.com/api/v4/projects/…/merge_requests/<n>/merge: 405 {message: 405 Method Not Allowed}
```

**ไม่ใช่เรื่องสิทธิ์** — โปรเจกต์ตั้ง "Pipelines must succeed" ไว้ ดังนั้น 405 แปลว่า
**pipeline ของ MR นั้นยังไม่ success** (กำลังรัน หรือยังไม่ถูกสร้าง เพราะเพิ่ง push ไปไม่กี่วินาที)

เจอ 3 ครั้งใน session เดียว (2026-09-13) กับ member FE !75, backoffice FE !604, member-api !120

**How to apply:** เจอ 405 ให้หา pipeline ของ MR นั้นแล้วรอจนไม่ใช่ running/pending/created
ก่อนสั่ง merge ใหม่ — อย่าไปแก้ permission หรือเปลี่ยนวิธี merge

```bash
pid=$(rtk proxy glab ci list -R $R --per-page 5 | grep "merge-requests/<n>" | head -1 | grep -oE '#[0-9]{5}' | tr -d '#')
for i in $(seq 1 25); do st=$(rtk proxy glab ci get -R $R -p "$pid" | grep -E '^status:' | awk '{print $2}');
  case "$st" in running|pending|created) sleep 20;; *) break;; esac; done
```

ถ้า pipeline success แล้วยังขึ้น 405 ค่อยไปดูเหตุอื่น (Draft, conflict, approval rule) ·
เมื่อ pipeline เสร็จพอดีตอนสั่ง glab จะพิมพ์ `✓ Pipeline succeeded.` ก่อน `✓ Merged!` เอง

เชื่อม [[reference_rtk_git_output_filtering]] [[reference_glab_ci_status_stale_pipeline]]
