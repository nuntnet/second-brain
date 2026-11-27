---
name: reference-console-prod-unit-job-flaky-on-prod-runner
description: "console production tag pipeline: unit job slow/flaky on the prod runner (fixed tests + 1h45m timeout in v1.2.7), shares the `test` resource_group with main's staging tests (blocks prod at night), deploy_production is MANUAL, clear_cache.py crashes"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-27T17:10:50.112Z
---

`frontend/ai-chat-admin-frontend` production tag pipeline (`unit_test_production` → build → `deploy_production`):

- **Slow/flaky unit run** on the production runner pool. 2026-11-17 v1.0.4 needed 4 runs; 2026-11-27 v1.2.4→v1.2.6 never deployed (5 failures, each different). Fixed for good in **v1.2.7** (2026-11-28): CasePanel countdown test freezes `Date` (`d2e461f2`), testing-library `asyncUtilTimeout` 15 s when `CI` (`f6640d2f`), `unit_test_production: timeout 1h 45m` + retry on `runner_system_failure` only (`64657390`, MR !197 → main). The run then took 47.5 min.
- **Shared lock:** the SRE template puts `unit_test_staging` (main pipeline, staging-th runner) and `unit_test_production` in the same `resource_group: test`. At night staging-th is off, a main test job sits pending holding the lock, and the prod tag waits (`waiting_for_resource`). Cancelling the pending main job (296261) released it.
- **`deploy_production` is manual** — the pipeline stops there after build. A watcher must treat `manual` as a terminal state, and someone must press play (that is the prod deploy, so it needs nunt's release OK).
- **Bad node:** `ip-10-111-11-43` intermittently fails pod start (`runc … unable to freeze`) → `runner_system_failure`; retry by job id lands elsewhere.
- **`clear_cache.py` after_script crashes** (`TypeError` on CloudFront `Aliases` list); the job still succeeds. Verify prod by fetching chat.sellsuki.com and matching `assets/index-*.js` against the job log, then grep the bundle for the release's new i18n keys.
- SRE request with all four: `docs/release/sre-request-production-runner-2026-11-27.md` (not yet sent as of 11-28).

**How to apply:** for a console prod release, retry runner failures once by id, check `waiting_for_resource` for a pending main job holding `test`, play `deploy_production` when the release is approved, and verify on the real site. Cut hotfix tags from the deployed/tested tag, not from `main`.
