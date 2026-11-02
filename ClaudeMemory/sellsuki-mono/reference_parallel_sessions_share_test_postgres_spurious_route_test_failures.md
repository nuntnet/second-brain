---
name: reference_parallel_sessions_share_test_postgres_spurious_route_test_failures
description: "Sibling sessions running chat-core `go test ./... -p 1` at the same time share local Postgres schemas (test schema names are fixed per package) → spurious failures in packages you didn't touch; re-run the failing packages alone and on origin/main before calling them yours or pre-existing"
metadata:
  type: reference
---

Seen 2026-11-02 (AI-393): chat-core full suite 7286 pass / 4 fail while AI-382 and AI-385 sessions ran chat-core
suites on the same `chat_core_test` DB (`ps aux | grep "go test"` showed three worktrees). Failures were in
`route/lead_reminder` (TwoCompaniesTwoWorkspacesNoLeakage, expected 201) and `route/workspace` (PatchAIConfig secret,
`workspace_not_accessible`) — packages the branch never touched. All passed when re-run alone; lead_reminder also
passed on a detached `origin/main` worktree. `internal/testdb.Open(t, schema)` uses a fixed schema per package, so
two checkouts of the same repo truncate each other's tables.

**How to apply:** before reporting a red full suite, check `ps aux | grep "[g]o test"` for other worktrees; re-run
just the failing packages; if still red, run them on `origin/main` to separate pre-existing from yours. Record the
re-run in the ledger's environment log instead of silently ignoring the red. Related:
[[reference_chat_core_db_tests_need_p1]].
