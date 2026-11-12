---
name: feedback_never_pkill_pattern_kill_by_pid_on_macos
description: "A verifier's `pkill -f \"bin-head-server\" -n` on macOS killed Docker Desktop, Claude, Codex, Dropbox — BSD pkill reads a trailing -n as a second PATTERN; agents must kill their own processes by recorded PID only"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 300c6962-12fa-4740-9aa3-3eb3ab997d45
  modified: 2026-11-12T06:03:15.269Z
---

On 2026-11-12 03:08, a BOLA-277 verifier agent ran `pkill -f "bin-head-server" -n` to stop its own server. On macOS (BSD pkill), options after the pattern are not options: `-n` became a second pattern. Every process whose command line contained "-n" was signalled: Docker Desktop (the daemon and every shared `sellsuki_mono-*` container), Claude, Codex, Grok, Adobe CC and Dropbox helpers.

Recovery: `open -a Docker` (needs unsandboxed), then `docker start sellsuki_mono-kafka-1 sellsuki_mono-mailslurper-1`, which exit 255 after a daemon kill and do not come back on their own.

**Disk-full hang (2026-11-12, nunt asked me to fix it):** with 3+ agents each running a Postgres container, the disk hit 100% and the Docker daemon hung (`docker ps` never returned, even after space was freed to 15 GB). `osascript -e 'quit app "Docker"'` did nothing in 2 min. What worked: confirm the exact PIDs with `ps -p <pid> -o comm=` (`Docker Desktop` and both `com.docker.backend`), `kill` them, `kill -9` the one that stays, then `open -a Docker` (up in ~6 s). Leave `com.docker.vmnetd` alone. Afterwards remove the agents' leftover test containers by name and `docker start` kafka + mailslurper. Prevent it: at most 2 Postgres-running agents at once, and check `df -h /System/Volumes/Data` before starting more.

**Why:** a pattern kill is a guess about which processes match, and on BSD a misplaced flag widens the guess to the whole machine.

**How to apply:** every brief to an agent that starts servers or containers says: record the PID when you start a process (`$!`) and stop only that PID (`kill <pid>`); never `pkill` / `killall` / `pkill -f`. For containers, use `docker rm -f <your own container name>`. Related: [[reference_ladder_survives_network_drop_check_before_restart]].
