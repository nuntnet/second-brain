---
name: oc2plus-ci-outage-2026-09
description: OC2Plus GitLab CI has been red for every MR since 2026-09-05 — first no runner (stuck_or_timeout), then Staging-th runner up but its registry mirror cannot pull golang:1.24 (ImagePullBackOff). Not code.
metadata:
  type: reference
---

Every OC2Plus pipeline (backoffice-api, member-api, 3rdparty-api, both FEs) has failed since
2026-09-05 for infra reasons, so a red MR pipeline in that window says nothing about the code:

- 09-05 → 09-08: `failure_reason=stuck_or_timeout_failure`, `runner=None` — no runner picked the
  `unit_test_merge_request*` job up (see [[dead-staging-runner-tag]]).
- 09-09: "Staging-th Runner General" is back but `registry.fountain.sellsuki.com/dockerhub/library/golang:1.24`
  fails with ImagePullBackOff / DeadlineExceeded → `runner_system_failure`. Retrying the job
  (`glab api -X POST projects/:id/jobs/<id>/retry`) reproduces it — the mirror, not the job.
- 09-09 16:08Z → 21:00Z: 9 MR pipelines (backoffice-api/FE/CCS3/bola) sat `stuck_or_timeout_failure`
  runner=None for ~5h, then on 09-10 retry the runner picks them up but the mirror
  `registry.fountain.sellsuki.com` answers `TLS handshake timeout` for alpine:3.15 / node:20 /
  golang → `runner_system_failure` after ~25s. Intermittent: some retries pass. Pipeline-level retry
  (`pipelines/:id/retry`) reruns only failed jobs and keeps skipped ones `created` — fine.

**How to apply:** a poll loop that retries `runner_system_failure`/`stuck_or_timeout_failure` jobs
up to ~4× (60s cadence) rides out the intermittency; only `script_failure` means look at code.
 read `failure_reason` and the first WARNING lines of the trace before touching
code; report "CI infra" explicitly. Backoffice-api / member-api / FE MRs use `ci_must_pass`, so
nothing there can merge until the mirror is fixed; 3rdparty-api MRs show `mergeable` (no CI gate)
but should not be merged on a red pipeline knowingly. Check whether it is fixed with the latest
pipeline on any branch: `glab api "projects/:id/pipelines?per_page=5"`.

## 2026-09-10 22:26–23:30+ Thai — ล่มอีกรอบ ต้นเหตุ = runner ตัวเดียวที่มี tag `staging-th` offline

**วิธีวินิจฉัยที่เร็วที่สุด (ทำซ้ำได้):**

```bash
glab api "runners/5153" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['description'], d['online'], d['tag_list'])"
```

**id 5153 = `Staging-th Runner General`, tags `['general','docker','sellsuki','amd','staging-th']`** — เป็น **runner ตัวเดียว** ที่มี tag `staging-th` · ทุก job ของ project 529 ต้องการ tags `['amd','docker','general','sellsuki','staging-th']` ครบชุด · runner อื่นที่ online (5125 `Production Private Gitlab Ru`, 5127 `Production Gitlab Runner`) มี tag `production` ไม่มี `staging-th` → **รับงานแทนไม่ได้**

วันนั้น pipeline 58187 เขียวปกติที่ 13:26 UTC บน runner ตัวนี้ แล้วมันหลุด offline ช่วง 13:26–15:26 UTC · ผลคือ pipeline ค้าง **7 ตัวพร้อมกัน**

### แยกสองสถานะให้ออก ก่อนจะไปแก้ผิดที่

| status | ความหมาย | ทางแก้ |
| --- | --- | --- |
| `pending` | job แรกของ pipeline **หา runner ไม่ได้** | ไม่มีอะไรทำได้ รอ runner กลับ |
| `waiting_for_resource` | ติดคิว resource_group ที่ pipeline เก่ากว่าถืออยู่ | ปกติคือ cancel ตัวที่ถือ ([[reference_stuck_ci_job_holds_resource_group]]) |

**แต่ถ้าตัวที่ถือ lock เป็น `pending` เพราะไม่มี runner → cancel ไม่ช่วยเลย** เพราะ pipeline ถัดไปก็จะ pending ต่อ · วันนั้น 58225 (develop, จาก push ที่ merge !39) เป็น `pending` แล้ว 5 ตัวหลังเป็น `waiting_for_resource` เรียงกัน — ถ้าดูแค่ status ของตัวหลังจะเข้าใจผิดว่าเป็น lock แล้วไป cancel เปล่า ๆ

**เช็ค tags ของ job ที่เคยสำเร็จ vs ที่ค้าง เป็นตัวชี้ขาด** — ถ้าเหมือนกันเป๊ะและ runner name เดิม แปลว่าไม่ใช่ config ของ MR แต่เป็น infra
