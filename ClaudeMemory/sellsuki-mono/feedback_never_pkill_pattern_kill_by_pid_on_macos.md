---
name: feedback_never_pkill_pattern_kill_by_pid_on_macos
description: "A verifier's `pkill -f \"bin-head-server\" -n` on macOS killed Docker Desktop, Claude, Codex, Dropbox — BSD pkill reads a trailing -n as a second PATTERN; agents must kill their own processes by recorded PID only"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 300c6962-12fa-4740-9aa3-3eb3ab997d45
  modified: 2026-11-11T20:13:24.689Z
---

On 2026-11-12 03:08, a BOLA-277 verifier agent ran `pkill -f "bin-head-server" -n` to stop its own server. On macOS (BSD pkill), options after the pattern are not options: `-n` became a second pattern. Every process whose command line contained "-n" was signalled: Docker Desktop (the daemon and every shared `sellsuki_mono-*` container), Claude, Codex, Grok, Adobe CC and Dropbox helpers.

Recovery: `open -a Docker` (needs unsandboxed), then `docker start sellsuki_mono-kafka-1 sellsuki_mono-mailslurper-1`, which exit 255 after a daemon kill and do not come back on their own.

**Why:** a pattern kill is a guess about which processes match, and on BSD a misplaced flag widens the guess to the whole machine.

**How to apply:** every brief to an agent that starts servers or containers says: record the PID when you start a process (`$!`) and stop only that PID (`kill <pid>`); never `pkill` / `killall` / `pkill -f`. For containers, use `docker rm -f <your own container name>`. Related: [[reference_ladder_survives_network_drop_check_before_restart]].
