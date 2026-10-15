---
name: reference_bola_postgres_tests_need_disposable_container_and_two_dsn_forms
description: bola-backend Postgres-gated tests: run against a throwaway postgres container on :55432, and the repo uses TWO DSN forms (URL vs key=value) depending on the package
metadata:
  type: reference
---

`BOLA_TEST_POSTGRES_DSN` gates the real-Postgres sub-tests (they create a schema per test, so any empty DB works). Do not point them at the dev DBs:
`docker run -d --name bola-test-pg273 -e POSTGRES_PASSWORD=bolatest -e POSTGRES_DB=bola_test -p 55432:5432 postgres:16-alpine`, then `docker rm -f` it afterwards.

**The DSN form differs per package** (pre-existing inconsistency, found 2026-10-14): `contact_profile_repository` appends `?search_path=` so it needs the URL form `postgres://postgres:bolatest@127.0.0.1:55432/bola_test?sslmode=disable`; `contact_metrics_repository` appends ` search_path=` so it needs `host=127.0.0.1 port=55432 user=postgres password=bolatest dbname=bola_test sslmode=disable`. A wrong form fails every Postgres sub-test with "failed to configure TLS (sslmode is invalid)" which looks like a code failure but is not. Run the two packages separately.

Race tests (e.g. BOLA-273 AC5 concurrent merge) are detectors, not deterministic: with the row lock removed they failed 7/10 runs. Prove red by looping, and do not count a mutation that breaks the SQL syntax.
