---
name: reference_kill_zombies_reads_proc_noop_on_macos
description: "scripts/kill-zombies.sh (make clean-procs) scans /proc, which macOS does not have — its orphan tmp/main and stale-air passes silently do nothing on this machine"
metadata:
  type: reference
---

Found 2026-09-20. `scripts/kill-zombies.sh` loops `for proc_dir in /proc/[0-9]*/` and tests `[ -d "/proc/$ppid" ]` / `readlink /proc/$pid/cwd`. On Darwin there is no `/proc`, so sections 2 (orphaned `tmp/main`) and 3 (stale `air`) never match anything, and `make clean-procs` reports success having killed nothing. CLAUDE.md's "port conflict → make clean-procs" advice is therefore empty here.

**How to apply:** on this Mac, find a port holder with `lsof -ti tcp:<port> -sTCP:LISTEN` and read its cwd with `lsof -a -p <pid> -d cwd -Fn` (that is how the 5-day-old backoffice-api binary was found). Do not kill service processes directly (overmind rule) — tell the user. The script itself is a small fix not yet made.

Also learned the same day: after a branch switch, air may take a minute or more to rebuild a large Go service — I called it "wedged" after ~30s and was wrong. Check the `tmp/main` mtime again before concluding.

Related: [[reference_overmind_dead_processes_are_a_wedged_session]] · [[reference_stray_claude_dev_server_squats_port]]
