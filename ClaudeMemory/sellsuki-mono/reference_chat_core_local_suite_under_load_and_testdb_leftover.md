---
name: reference_chat_core_local_suite_under_load_and_testdb_leftover
description: "chat-core full local suite times out under machine load (re-run failed pkgs with -p 1, ./test with -timeout 60m); internal/testdb leaves a <db>_tp_<hash> DB to drop too"
metadata:
  node_type: memory
  type: reference
  originSessionId: cc30fdb9-527c-4eb2-92a5-823515ceebc9
  modified: 2026-11-20T11:29:45.994Z
---

On 2026-11-20 (AI-500) the chat-core full `go test ./src/...` with `CHAT_CORE_TEST_POSTGRESQL_URI` hit 10m package timeouts and fiber `test: timeout error 1000ms` in 16 DB-heavy packages (route/workspace, case_, lead*, repository/*). `./test/...` hit its 30m limit too. The machine's load average was ~30 from parallel sessions. None of it was a real failure: re-running only the failed packages with `-p 1 -timeout 40m`, and `./test/...` with `-timeout 60m` (~25 min), was all green.

**How to apply:** collect the FAIL package list from the log and re-run just those serially before calling anything red. Check `uptime` first.

Cleanup trap: after `DROP DATABASE <yours> WITH (FORCE)`, a second database named `<yours>_tp_<hash>` stays behind (internal/testdb's per-binary copy). List it with `select datname from pg_database where datname like '<yours>%'` and drop that one as well.

Related: [[feedback_parallel_wave_sessions_cap_agents_and_ladders]]
