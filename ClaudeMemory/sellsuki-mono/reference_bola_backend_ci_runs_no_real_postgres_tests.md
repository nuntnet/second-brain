---
name: reference-bola-backend-ci-runs-no-real-postgres-tests
description: "bola-backend CI never runs the real-Postgres tests: INTEGRATION_TEST_SCRIPT is echo \"1 + 1\", unit job is go test ./src/... only; tests under test/ or gated by BOLA_TEST_POSTGRES_DSN skip, so a green pipeline proves nothing about them (checked 2026-11-11)"
metadata:
  type: reference
---

Checked on `backend/bola-backend` `origin/main` 2026-11-11 (`.gitlab-ci.yml` lines 19-29):
`UNIT_TEST_SCRIPT: 'go test ./src/... -p 4'`, `INTEGRATION_TEST_SCRIPT: 'echo "1 + 1"'`, `E2E_TEST_SCRIPT: 'echo "1 + 1"'`.

So every test under `test/` (BOLA-383 `test/contact_api`, BOLA-357 `test/contact_profile_eager`, ...) and every `src/` test gated by `BOLA_TEST_POSTGRES_DSN` (BOLA-273 merge race, contact_metrics, contact_profile custom fields) never runs in CI. They skip silently when the DSN is unset, which looks green.

**How to apply:** for a bola-backend card whose proof is a real-Postgres test, the gate is the local run on Postgres (:5432, see BOLA-383 ledger Gotchas) plus a verifier verdict; say in the MR description that CI does not run them. This is the repo's norm, not a broken pipeline; changing it is a `refactor/` CI card cut from `main` first (repos.md Git flow §7). Related: [[reference_bola_test_api_broadcast_mock_panic_on_main]].
