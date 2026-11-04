---
name: reference_bola_test_api_broadcast_mock_panic_on_main
description: "BOLA `go test ./test/api/` panics on main (MockBroadcastRepository missing CountBroadcasts expectation) — pre-existing, not your branch"
metadata:
  node_type: memory
  type: reference
  originSessionId: fdaedea2-7793-4a09-8cbe-7ddef0766886
  modified: 2026-11-04T17:00:27.197Z
---

Verified 2026-11-04 on bola-backend `origin/main` b2d0f47: `test/api` panics with `mock: I don't know what to return … CountBroadcasts` from `use_case/broadcast.go` ListBroadcasts. Every other package (70) passes. A feature branch cut from main shows the same failure.

**How to apply:** when BOLA's full suite shows only this failure, compare against main once (detached worktree) and report it as baseline, not a regression. Needs a fix card of its own if nobody has one.
