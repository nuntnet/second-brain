---
name: reference-chat-core-storage-coverage-counts-in-package-tests-only
description: "chat-core CI storage_coverage (70% gate) runs go test -coverprofile ./src/... WITHOUT -coverpkg — ./test/ integration tests add nothing; an external _test package in the same dir does count"
metadata:
  node_type: memory
  type: reference
  originSessionId: 591693eb-81b7-4658-8ca1-8f3d48d02730
  modified: 2026-11-05T08:42:06.868Z
---

chat-core's `storage_coverage` CI job runs `go test -coverprofile cover.out -p 1 ./src/...` with no `-coverpkg`, so a package only gets credit for tests that live in its own directory. The big integration suites under `./test/` (board, customer_list, ai376 dashboard, ai365 welcome sort…) contribute zero. Threshold 70% total; wave-2 MR !206 fell to 65.9% because new use-case packages had all their tests in ./test/ (fixed 2026-11-05 by in-package tests, +1,572 stmts in one set).

- An external test package (`package foo_test`) in the same directory DOES count — use it when an in-package test would create an import cycle (case_repository ↔ lead_repository migrations).
- Measure per package exactly like CI: `go test -count=1 -p 1 -coverprofile=<f> ./src/<pkg>/`, no -coverpkg.
- When adding a feature, put at least the AC tests in the package itself, not only in ./test/.

Related: [[reference_chat_core_db_tests_need_p1]], [[feedback_tests_derive_from_the_card_not_the_code]]
