---
name: reference-local-duplicate-overmind-chat-core-and-procfile-reload
description: "Local chat-core can run twice (stray .overmind-chat-core.sock instance + ai-mvp stack) and the duplicate silently consumes the inbound queue; `overmind restart` never rereads Procfile.ai-mvp — use scripts/ai-mvp-restart.sh"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-18T13:24:02.909Z
---

2026-11-18, found by the AI-470 verifier: a second overmind instance (`.overmind-chat-core.sock`, started the night before) ran chat-core without the ai-mvp env. It bound no port (8099 was taken) but kept pulling queued customer messages, so half the turns skipped the budget gate (no REDIS_URL). `make clean-procs` did NOT remove it (its air parent was alive). Fix: `overmind stop chat-core -s .overmind-chat-core.sock` then `overmind quit -s .overmind-chat-core.sock`.

Also: after editing `Procfile.ai-mvp`, `overmind restart <name> -s .overmind-ai-mvp.sock` keeps the OLD command line. Only `scripts/ai-mvp-restart.sh` (quits and restarts the whole socket, ~1–4 min) picks up the edit. Check with `ps eww -p <pid> | tr ' ' '\n' | grep '^VAR='`.

**How to apply:** before trusting a local verifier run, check for more than one `backend/sellsuki-chat-core/tmp/main` process and list `.overmind*.sock`. Related: [[reference-overmind-start-d-does-not-start-one-process-use-restart]].
