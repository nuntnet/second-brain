---
name: reference_overmind_start_d_does_not_start_one_process_use_restart
description: "\"overmind start <name> -D -s <sock>\" tries to launch a NEW overmind instance (refused: \"already running\") — it does not start one stopped process; after `overmind stop <name>` only `overmind restart <name>` brings it back"
metadata:
  node_type: memory
  type: reference
  originSessionId: 17b79f90-4e6f-4d55-85bf-48f14625212a
  modified: 2026-11-14T14:51:33.888Z
---

2026-11-14 (AI-442, ai-mvp stack, socket `.overmind-ai-mvp.sock`): to pick up a new env line I ran `overmind stop messaging` then `overmind start messaging -D -s .overmind-ai-mvp.sock` as `.claude/rules/overmind.md` says. The start was refused ("it looks like Overmind is already running") and messaging stayed **dead** — other sessions lost messaging until I ran `overmind restart messaging -s .overmind-ai-mvp.sock`, which brought one process back on :8100 (no duplicate).

**How to apply:** to restart one service in a running overmind, use `overmind restart <name> -s <sock>` and check `overmind status` + `lsof -iTCP:<port> -sTCP:LISTEN` shows exactly one listener. The rule file's "stop then start -D" does not work for a single process. Related: [[reference_ai_mvp_restart_messaging_dies_if_file_service_not_up]].
