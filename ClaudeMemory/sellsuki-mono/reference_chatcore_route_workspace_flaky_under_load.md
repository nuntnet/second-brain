---
name: chatcore-route-workspace-flaky-under-load
description: "chat-core's route/workspace package fails a DIFFERENT test each full-repo run and passes alone — Postgres contention from MigrateUp per test, not a logic bug"
metadata:
  type: reference
---

Observed three times, 2026-08-30/31, on `backend/sellsuki-chat-core`.

**Symptom:** `go test ./src/... ./cmd/...` fails one test in
`src/interface/fiber_server/route/workspace`, and it is a **different test each
run** — seen so far:
`TestPatchGuardrailConfig_RejectsDisclosureTemplatesWithoutDefault`,
`TestPatchGuardrailConfig_RequiresVersion`,
`TestPatchGuardrailConfig_StaleVersionIsAConflict`.

**It is not a real failure.** Alone the package passes in ~43–47 s; under
full-repo load it takes 129–245 s and one test tips over. The named tests
contain **no timing assertion** and the package has **no `t.Parallel()`**, so
there is no intra-package race, and `testdb.Open` gives each package its own
schema so there is no cross-package truncation race either.

**Mechanism:** every one of the package's ~45 tests calls `newHarness` →
`testdb.Open` + `workspace_repository.MigrateUp` + a multi-table `TRUNCATE`
against a shared Postgres. Under `go test ./...` many packages hit that server
at once and the per-test migrate/truncate becomes the bottleneck.

**Do not "fix" it by retrying.** Confirm with:
```bash
go test ./src/interface/fiber_server/route/workspace/ -count=1        # passes, ~45s
go test ./src/interface/fiber_server/route/workspace/ -run '<TheTest>$' -count=1   # passes, ~1s
```
The real fix is running `MigrateUp` once per package instead of once per test
(truncate per test still provides the isolation). Not done yet — it is a
rewrite of someone else's harness, and worth doing with the failure captured.

**A different flaky in the same repo WAS fixed** (MR !48):
`TestUseCase_FetchContextSources_RunsInParallel` asserted wall-clock time and
now asserts observed concurrency via a rendezvous. See
[[timing-dependent-concurrency-tests]].

Related: [[chatcore-route-tests-timeout-under-load]],
[[chatcore-ci-gaps]].
