---
name: reference_bola_postgres_tests_need_disposable_container_and_two_dsn_forms
description: bola-backend Postgres-gated tests: throwaway postgres on :55432 (bola-test-pg273 usually already running), TWO DSN forms, and the sandbox/worktree/uuid traps that fake a red or a green (2026-10-24)
metadata:
  type: reference
---

`BOLA_TEST_POSTGRES_DSN` gates the real-Postgres sub-tests (they create a schema per test, so any empty DB works). Do not point them at the dev DBs:
`docker run -d --name bola-test-pg273 -e POSTGRES_PASSWORD=bolatest -e POSTGRES_DB=bola_test -p 55432:5432 postgres:16-alpine`, then `docker rm -f` it afterwards.

**The DSN form differs per package** (pre-existing inconsistency, found 2026-10-14): `contact_profile_repository` appends `?search_path=` so it needs the URL form `postgres://postgres:bolatest@127.0.0.1:55432/bola_test?sslmode=disable`; `contact_metrics_repository` appends ` search_path=` so it needs `host=127.0.0.1 port=55432 user=postgres password=bolatest dbname=bola_test sslmode=disable`. A wrong form fails every Postgres sub-test with "failed to configure TLS (sslmode is invalid)" which looks like a code failure but is not. Run the two packages separately.

Race tests (e.g. BOLA-273 AC5 concurrent merge) are detectors, not deterministic: with the row lock removed they failed 7/10 runs. Prove red by looping, and do not count a mutation that breaks the SQL syntax.

**Traps found on BOLA-380 (2026-10-24):**
- `bola-test-pg273` (password bolatest, db bola_test) is usually already up on :55432 from an old session; `docker run` of a second one fails on the port. Reuse it (tests use a per-test schema and drop it); remove only a container you created.
- The sandbox blocks both the docker socket and loopback dials (`operation not permitted`), so a Postgres subtest "fails" without running. Run those `go test` / `docker` commands with the sandbox off, and set `GOCACHE=$TMPDIR/gocache` or the build cache write fails.
- `admins.id` / `workspace_id` are `uuid` columns: fixtures with ids like "a1" fail on Postgres with SQLSTATE 22P02 but pass on SQLite. Before calling a Postgres red proof, read the message for the real symptom, not just FAIL.
- A submodule worktree cannot be added under the main checkout path (sandbox write limit); add it in the session scratchpad. The monorepo worktree's `backend/bola-backend` is an empty dir holding a harness `.claude/.cc-writes` — leave it.
- Pre-push secret scanner blocks a commit containing a `postgres://user:pass@…` URL even in a test comment; describe the DSN form without the password.
