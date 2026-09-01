---
name: reference_coverage_gate_on_dbless_job
description: A Go coverage gate wired into E2E_TEST_SCRIPT runs with no database, so DB-backed tests skip and the total is a lie — still latent in space-go and sellsuki-ai-agent
metadata:
  type: reference
---

The shared SRE `pipeline-deployment` template gives its `e2e_test_*` job **no
`services:`**, so `E2E_TEST_SCRIPT: "make check-coverage"` measures `./src/...`
on a runner with no Postgres. Every DB-backed test calls `testdb.Open`, which
`t.Skip`s, and the total collapses.

Measured on `backend/sellsuki-chat-core` (2026-09-02):

| | packages at `coverage: 0.0%` | total |
|---|---|---|
| CI (no DB) | 21 | **46.5%** |
| local, DB attached | 5 | **72.1%** |
| local, `CHAT_CORE_TEST_POSTGRESQL_URI` → dead port | 21 | **46.1%** ← reproduces CI |

**Two things make this look like a real coverage failure instead of a missing
database:**

1. **`t.Skipf` output only prints under `-v`**, and `coverage-test` has no `-v`.
   Grepping the failed job trace for the skip message returns **zero matches** —
   which reads as proof that nothing skipped. It is not. Count
   `coverage: 0.0% of statements` lines and compare against a local run instead.
2. The job runs on `main`, so it takes the deploy job down with it. The visible
   symptom is a blocked staging deploy, not a test problem.

**Do not lower `threshold` to make it pass.** That retires the gate while leaving
it looking alive. Fix: move `make check-coverage` onto the job that declares the
Postgres/Redis services (in chat-core that is `integration_test`, and it needed
`- if: '$CI_COMMIT_BRANCH == "main"'` added so the gate does not vanish from the
deploying branch). Also add `-p 1` to `coverage-test` — `./src/...` includes
`./src/repository/...`, whose tests each run `MigrateUp` against one shared
Postgres; concurrent package binaries are the mechanism behind
[[reference_chatcore_route_workspace_flaky_under_load]]. Landed as chat-core MR !51.

**Still latent elsewhere.** Both of these have `E2E_TEST_SCRIPT: "make check-coverage"`
with the same template and no DB service of their own:
- `backend/space-go`
- `backend/sellsuki-ai-agent`

`backend/pis-api` avoids it by putting the gate in `UNIT_TEST_SCRIPT` and only
`make coverage-test` (no gate) in E2E; `backend/rag-core` avoids it by scoping
coverage to `src/entity` and `src/use_case`, which need no database.
