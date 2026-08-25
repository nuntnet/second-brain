---
name: messaging-backend-shared-repo-traps
description: "sellsuki-messaging-backend is a shared OTP/SMS service — .env is tracked in git, CI is switched off, and develop (once stale) is now the live line"
metadata:
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-25T11:29:59.057Z
---

`backend/sellsuki-messaging-backend` is a **shared platform service** (OTP/SMS for the whole org — see [[messaging-backend]]) that the AI chat work bolted a chat module onto. Three traps, all found 2026-08-11 while preparing the merge:

**1. `.env` is tracked in git and is NOT gitignored.** It is shared state, not a local file. Editing it for local dev and committing silently repoints *everyone else's* config on their next pull. The AI MVP branch had overwritten nine pre-existing values — `POSTGRES_HOST/PORT/USER/PASS/DB_NAME_MESSAGING`, `ORY_KRATOS_ADMIN_SERVER`, `ORY_KETO_READ_GRPC_SERVER`, and both `THAIBULK_OTP_BASEURL` / `THAIBULK_SMS_BASEURL` (the OTP/SMS vendor endpoints). **No key was removed** — the set was a superset — so nothing looked broken from our side; only a value-by-value comparison catches it. Always diff `.env` values against the target branch before merging here.

**2. `develop` vs `main` — this flipped, re-check before every merge.** As of 2026-08-11 develop was stale (last commit 2026-03-02, main 9 ahead). **By 2026-08-25 it had reversed: `develop` is 61 commits ahead of `main`, and main is only 1 ahead of develop** (both took OC-3896 on 2026-08-19). So develop is now the live line, matching the stated policy. Practical consequence: the AI-chat branch `feature/mvp-media-and-secret-durability` (MR !14) targets `main` and reads as 57 ahead / 17 behind there, but only **10 ahead / 30 behind `develop`** — retargeting to develop is the smaller, correct merge. Always measure both with `git rev-list --count` rather than trusting either this note or the policy.

**3. There is no CI.** The SRE deployment pipeline `include:` is commented out in `.gitlab-ci.yml` and only `.variables_export_*` anchors remain — **no build/test/deploy job runs on an MR here**. Local verification is the only gate: `go build ./...`, `go vet ./...`, `make unit-test` (the OTP suite is the canary that the chat module hasn't disturbed the original service).

Also pre-existing, worth a card: `src/interface/fiber_server/helper/metrics.go:92` uses the raw request path as a Prometheus label (`strings.Replace(ctx.Path(), "/", "", 1)`), so every distinct id in a path becomes its own series. Harmless while routes were static; the FB routes added `:app_id`.

Chat-module isolation is real and guarded, not just asserted: `src/interface/fiber_server/route_isolation_guard_test.go` (static scan of the whole package for stray `/v1/chat` routes) + `test/isolation/db_isolation_test.go` (separate schema).
