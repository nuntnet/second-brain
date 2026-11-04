---
name: reference_ladder_survives_network_drop_check_before_restart
description: A background test ladder that looks dead after a session network drop (ENOTFOUND) may still be running; restarting it puts two runs on one test DB → mass bogus FAILs
metadata:
  node_type: memory
  type: reference
  originSessionId: fdaedea2-7793-4a09-8cbe-7ddef0766886
  modified: 2026-11-04T17:00:26.832Z
---

On 2026-11-04 (V2-5, chat-core) the session dropped on ENOTFOUND, the background `go test -p 1` ladder was assumed dead and restarted. The first one was still alive; both finished and appended to the same output file, and ~40 packages failed with cross-talk (extra rows, 500s, nil panics) on the shared `chat_core_test_*` DB.

**How to apply:** before restarting a ladder after a drop, check its done-file / task notification or `pgrep -f "go test"` (unsandboxed — `ps` is blocked in the sandbox). Write each run to a fresh output file plus a `.done` marker so two runs can't merge. Also: a nohup script launched from sandboxed Bash stays sandboxed → every httptest bind fails EPERM at 0.00s ([[reference_sandbox_bind_eperm_fails_go_tests_and_slog_panics_in_tests]]); launch it with the sandbox disabled. Related: [[reference_background_ladder_dies_with_the_turn_use_nohup]].
