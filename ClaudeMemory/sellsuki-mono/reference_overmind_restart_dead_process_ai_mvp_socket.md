---
name: reference_overmind_restart_dead_process_ai_mvp_socket
description: "Bringing back ONE dead process on the AI stack: `overmind restart -s /Users/nunt/sellsuki_mono/.overmind-ai-mvp.sock <name>` works; the rules' `overmind start <name> -D` (even with OVERMIND_SOCKET set) fails 'Overmind is already running'. Safe only when the process is already dead (no duplicate)."
metadata:
  node_type: memory
  type: reference
  originSessionId: 55f225da-3e5e-4b6f-9508-e655e398936e
  modified: 2026-10-29T15:29:12.928Z
---

Seen 2026-10-29: a verifier crashed local ai-agent. `OVERMIND_SOCKET=… overmind stop ai-agent` worked, but
`overmind start ai-agent -D` tried to start a whole new overmind ("it looks like Overmind is already running").
`overmind restart -s <ai-mvp socket> ai-agent` started it; ~60–90 s until :8101 listens (air build). `.claude/rules/overmind.md`
forbids `overmind restart` because of duplicates — that risk is for a RUNNING process; check `overmind status -s …` shows
`dead` first. Also: `scripts/doctor.sh` reads the default socket and reports "overmind not running" for the AI stack.

Related: [[reference_local_usage_routes_cannot_show_allow_cells]].
