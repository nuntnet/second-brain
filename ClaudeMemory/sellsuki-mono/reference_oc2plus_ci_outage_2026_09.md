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

**How to apply:** read `failure_reason` and the first WARNING lines of the trace before touching
code; report "CI infra" explicitly. Backoffice-api / member-api / FE MRs use `ci_must_pass`, so
nothing there can merge until the mirror is fixed; 3rdparty-api MRs show `mergeable` (no CI gate)
but should not be merged on a red pipeline knowingly. Check whether it is fixed with the latest
pipeline on any branch: `glab api "projects/:id/pipelines?per_page=5"`.
