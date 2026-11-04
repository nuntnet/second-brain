---
name: reference_background_ladder_dies_with_the_turn_use_nohup
description: a run_in_background test ladder can be killed (log truncated, no test lines) when the turn ends on an error; run long ladders as a nohup script that writes its own log + a done file
metadata:
  type: reference
---
On 2026-11-04 a turn ended on a network error mid-ladder: the `run_in_background` chat-core ladder's log kept only BUILD_OK/VET_OK and a bare `SRC_EXIT=1`, with no test output, so the result meant nothing. Re-run as `nohup script.sh &` where the script redirects each `go test` to its own log and finally touches a `.done` file; wait on the done file with a separate background `until [ -f … ]` watcher. Read the log before re-running anything. Related: [[reference_parallel_sessions_share_test_postgres_spurious_route_test_failures]].
