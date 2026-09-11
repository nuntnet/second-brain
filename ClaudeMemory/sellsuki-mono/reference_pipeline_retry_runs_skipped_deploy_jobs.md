---
name: reference_pipeline_retry_runs_skipped_deploy_jobs
description: "GitLab pipeline retry re-runs SKIPPED jobs, not just failed ones — retrying a red pipeline deployed a frontend to staging that I had promised not to deploy"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-11T05:33:46.690Z
---

`POST /projects/:id/pipelines/:pipeline_id/retry` re-runs **failed, canceled AND skipped** jobs. A manual deploy job that shows `skipped` because its upstream test stage failed **is in that set** — so retrying the pipeline to re-run flaky tests also ships the deploy.

**What it cost (2026-09-11).** Three repos had red staging pipelines from runner failures (`stuck_or_timeout_failure` / `runner_system_failure`), so I retried all three pipelines and told the user "retry only, ไม่ได้ deploy". In `sellsuki-invitation` the `deploy_staging` job was `skipped`, the retry ran it, and staging moved `ac1505cd` (11 Aug) → `f96a29c3` — **25 commits** across two teams' features. Proof it was the retry and not someone else: job 243227 was `created_at` 2026-09-09 but `started_at` 2026-09-11T04:06:57, minutes after the retry call.

**Do this instead:** retry the specific failed jobs by id — `POST /projects/:id/jobs/:job_id/retry` — after listing them with `.[] | select(.status=="failed") | .id`. Before any pipeline-level retry, check for deploy-stage jobs in a non-terminal state:

```
glab api "projects/$E/pipelines/$PID/jobs?per_page=60" \
  | jq -r '.[] | select(.stage=="deploy") | "\(.name)\t\(.status)"'
```

If any deploy job is `skipped`, `manual` or `created`, a pipeline retry may ship it. Say so before acting, and prefer per-job retries.

**Second lesson — a promise about scope has to match the mechanism.** I described the action correctly in intent ("retry, not deploy") but had not checked what the API actually does. When an action touches a shared environment, verify the mechanism's blast radius *before* describing it as safe, not after someone reports breakage.

Related: [[reference_stuck_ci_job_holds_resource_group]] (cancel vs retry for jobs stuck `pending` — a different case, and there retry is also the wrong move), [[reference_manual_staging_gate_silent_drift]] (why these deploy jobs sit skipped/manual for weeks in the first place, so the range a single retry ships is large).
