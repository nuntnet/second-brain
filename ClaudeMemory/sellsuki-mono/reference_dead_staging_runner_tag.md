---
name: reference-dead-staging-runner-tag
description: "CI jobs tagged 'staging' queue forever — that runner died ~Apr 2026; repos must include the -th SRE pipeline template variant"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-07T16:16:50.497Z
---

Runner tag **`staging` has no live runner** since the staging-th migration (~April 2026). Live runners: `staging-th` (5153 general/amd, 5154 arm, 5155 base) and `production`. Any repo still including the OLD SRE template `pipelines/gitlab-ci-pipeline.generic-frontend-npm.yml` (from `sellsuki/sre/deployment/pipeline-deployment`) produces jobs tagged `staging` that stay **pending forever** — no error, just silent queueing (sellsuki-invitation's main pipeline sat "running" from 6 Apr to 11 Aug unnoticed).

**Fix:** switch the include to the `-th` variant `gitlab-ci-pipeline.generic-frontend-npm-th.yml` (jobs extend `.staging_th_runner_amd_tags`; job structure otherwise identical — verified by job-name diff). Fixed for sellsuki-invitation in MR !15 (2026-08-11); sibling frontends like sellsuki-company-management-frontend already use it.

**⚠️ DIFFERENT failure, same look — the LIVE `staging-th` fleet itself went down (2026-09-04 ~14:00 →).**
Jobs correctly tagged `['amd','docker','general','sellsuki','staging-th']` also sat unpicked: every
`unit_test_*` in oc2plus backoffice-api AND member-api from 04 Sep 14:48 onward (MRs !538/!544/!98
and develop's own 57272) ended `failure_reason: stuck_or_timeout_failure`, `runner: null`,
`started_at: null` after GitLab's ~41 h queue timeout; last job that actually ran = 04 Sep 10:16 on
runner 5153. So do NOT "fix" the include here — the tag is right, the runners are gone. Diagnose with
the jobs API (`glab api projects/:id/pipelines/<id>/jobs` → `.status/.failure_reason/.runner.id`),
not `glab ci status` (which just says "failed"). `glab api jobs/<id>/trace` is 0 bytes for a job that
never started. `projects/:id/runners` returns null with this token (no permission to list).
Consequence: `only_allow_merge_if_pipeline_succeeds: true` blocks EVERY merge in the group until SRE
restores 5153/5154; a Developer token cannot arm merge-when-pipeline-succeeds. Workaround that keeps
work moving: prove green locally on the MR sha in a `git worktree add --detach <dir> origin/<branch>`
(`go build ./... && go test ./src/...`), then `POST merge_requests/<iid>/pipelines` to queue a fresh
pipeline that auto-runs the moment a runner returns; cancel zombie `pending` pipelines on closed MRs
(`POST pipelines/<id>/cancel`) so they don't sit ahead in the queue.

**Recurs for:** any dormant repo with the old include. Symptom signature = job pending with tags `['amd','docker','general','sellsuki','staging']` + zombie old pipelines stuck "running"/"waiting_for_resource" (cancel those to clear the queue).

**🔁 เกิดซ้ำ 2026-09-14 (~13:31 → อย่างน้อย 17:00)** — fleet `staging-th` ทั้ง 3 ตัว
(5153 general / 5154 arm / 5155 base) ออฟไลน์พร้อมกันอีกรอบ เหลือแต่ Production
runner 2 ตัวที่ไม่รับ tag นี้ · กระทบทุกรีโป OC2Plus ไม่ใช่รีโปเดียว (เช็คแล้วทั้ง
3rdparty-api, backoffice-api, backoffice FE เห็น online 2 / offline 3 เหมือนกัน)

อาการ: MR pipeline ค้าง `pending` โดย **ไม่มี runner ถูก assign เลย** (`runner: -`)
ต่างจากงานที่รันแล้วช้า · pipeline ของ MR ก่อนหน้าที่สำเร็จเมื่อ 10:58 ใช้
`Staging-th Runner General` ตัวเดียวกับที่ตอนนี้ตาย — นั่นคือวิธียืนยันว่าเป็นเรื่อง
runner ไม่ใช่เรื่อง tag ผิดหรือโค้ดพัง:

```
glab api "projects/:id/runners?per_page=10"        # ดูว่า online กี่ตัว
glab api "projects/:id/pipelines/<id>/jobs"        # runner=- + status=pending = ไม่มีใครรับ
glab api "projects/:id/pipelines/<last_success>/jobs"  # ตัวที่เคยรันใช้ runner ไหน
```

**อย่าทำ:** retry pipeline รัว ๆ (ไม่ช่วย งานยังเข้าคิวเดิม) · อย่า merge ข้ามด่าน CI ·
อย่าไปแก้ include template (tag ถูกอยู่แล้ว) · สิ่งเดียวที่ทำได้คือแจ้ง SRE ให้ปลุก fleet
