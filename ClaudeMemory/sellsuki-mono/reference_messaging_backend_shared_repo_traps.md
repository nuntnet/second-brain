---
name: messaging-backend-shared-repo-traps
description: "sellsuki-messaging-backend is a shared OTP/SMS service — .env is tracked in git, develop is stale, and CI is switched off"
metadata:
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-11T14:00:49.140Z
---

`backend/sellsuki-messaging-backend` is a **shared platform service** (OTP/SMS for the whole org — see [[messaging-backend]]) that the AI chat work bolted a chat module onto. Three traps, all found 2026-08-11 while preparing the merge:

**1. `.env` is tracked in git and is NOT gitignored.** It is shared state, not a local file. Editing it for local dev and committing silently repoints *everyone else's* config on their next pull. The AI MVP branch had overwritten nine pre-existing values — `POSTGRES_HOST/PORT/USER/PASS/DB_NAME_MESSAGING`, `ORY_KRATOS_ADMIN_SERVER`, `ORY_KETO_READ_GRPC_SERVER`, and both `THAIBULK_OTP_BASEURL` / `THAIBULK_SMS_BASEURL` (the OTP/SMS vendor endpoints). **No key was removed** — the set was a superset — so nothing looked broken from our side; only a value-by-value comparison catches it. Always diff `.env` values against the target branch before merging here.

**2. `develop` is stale, but policy still says target it.** Last commit 2026-03-02; `main` is 9 commits ahead (Careroms' CI/deploy MRs through April 2026). develop has no unique work — just older copies of `.gitlab-ci.yml`, `deployment/values-base.yml`, `deployment/values-staging.yml`. So merging a main-based branch into develop is purely forward, no revert, no conflict. Someone still has to promote develop→main later for anything to ship.

**3. There is no CI.** The SRE deployment pipeline `include:` is commented out in `.gitlab-ci.yml` and only `.variables_export_*` anchors remain — **no build/test/deploy job runs on an MR here**. Local verification is the only gate: `go build ./...`, `go vet ./...`, `make unit-test` (the OTP suite is the canary that the chat module hasn't disturbed the original service).

Also pre-existing, worth a card: `src/interface/fiber_server/helper/metrics.go:92` uses the raw request path as a Prometheus label (`strings.Replace(ctx.Path(), "/", "", 1)`), so every distinct id in a path becomes its own series. Harmless while routes were static; the FB routes added `:app_id`.

Chat-module isolation is real and guarded, not just asserted: `src/interface/fiber_server/route_isolation_guard_test.go` (static scan of the whole package for stray `/v1/chat` routes) + `test/isolation/db_isolation_test.go` (separate schema).
