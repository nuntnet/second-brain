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
