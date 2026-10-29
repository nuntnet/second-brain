---
name: reference_chat_core_db_tests_need_p1
description: "chat-core `go test ./src/...` against local Postgres in parallel fails with \"could not create file base/... File exists\" on TRUNCATE — run repository/route packages with -p 1; sandboxed runs also need GOCACHE in scratch and cannot bind ports"
metadata:
  node_type: memory
  type: reference
  originSessionId: c8122cba-dbd5-4605-8d20-45439f856a8f
  modified: 2026-10-29T05:51:10.819Z
---

Running the whole chat-core suite at once (`go test ./src/... ./cmd/...`) against the shared local test DB (`chat_core_test`) produced ~14 failures, all `truncate <table>: ERROR: could not create file "base/…": File exists`, plus knock-on route failures. Packages truncating the same DB concurrently caused it, not a code fault. `make repository-test` uses `-p 1` for this reason.

Also seen the same day: the system-wide fd limit ("Too many open files in system") broke git and rtk output while a big serial `go test` ran. Wait for it to finish before committing.

**`-p 1` is per process, not per machine.** On 2026-10-29 (AI-332), two `go test ./... -p 1` runs at once (mine plus a verifier subagent's, in different worktrees, same `chat_core_test` DB) failed 29 DB packages: "should have 1 item, has 3", 409 on fresh rows, wrong-company 403s. No code fault. When a verifier subagent runs its own full suite, don't run yours in parallel. Stop one of them, or rerun the failed packages alone afterwards.

Same again on 2026-10-29 (AI-334) while AI-213/AI-332 sessions ran their suites: 26 packages red with `column ... already exists` / `contains null values` (two processes migrating the schema at once), PLAYBOOK_VERSION_CONFLICT, a websocket receiving another test's message, Redis rate_limiter counts off. Check `ps aux | grep '\.test '` before trusting a full run.

**How to apply:** for the DB-backed slice run `go test -p 1 $(go list ./src/... | grep -E "/repository/|/route/")` outside the sandbox. In the sandbox: `GOCACHE=$TMPDIR/gocache` (the default cache is not writable), DB tests skip silently, and httptest servers panic on bind (e.g. AI-347's `fakeAgent`). See [[reference_sandbox_blocks_localhost_so_db_tests_skip_silently]], [[reference_tmpdir_differs_sandboxed_vs_unsandboxed_bash]].
