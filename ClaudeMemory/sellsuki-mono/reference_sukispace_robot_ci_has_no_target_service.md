---
name: reference-sukispace-robot-ci-has-no-target-service
description: testing/sukispace-api-automate-testing CI job `api-automate` runs the Robot suite against localhost:8080 with no space-go up → every MR pipeline shows 119/119 failed (connection refused); the red is not about the change
metadata:
  type: reference
---

Seen 2026-10-08 on MR !3 of `sellsuki/line-diy/test/line-diy-api-automate-testing` (pipeline 63191, job 265760): the
job installs requirements then runs Robot; every request dies with `HTTPConnection(host='localhost', port=8080) …
Connection refused` and the summary is `119 tests, 0 passed, 119 failed`. No service is started in the job and no
`BASE_URL` points at an environment. So a red pipeline on this repo says nothing about the MR; the suite only means
something when run locally against `make dev` (space-go on 8082 — note the suite's default is 8080, check the
variables file) or against dev with the right base URL.

**How to apply:** do not retry this job or chase it as a regression; say in the MR/ledger that CI here is non-signal
and report the local run instead. A card to give this job a target (compose service or env URL) belongs to the
repo-health list in `docs/cards/drafts/sukispace/OUTSIDE-SG.md`. See [[reference-backoffice-fe-ci-skips-all-tests]]
for the same shape on another repo.
