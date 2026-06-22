---
name: project_overmind_restart_quirk
description: "overmind — `overmind start <name> -D` does NOT work as CLAUDE.md claims; use `overmind restart <name>` for a dead proc"
metadata: 
  node_type: memory
  type: project
  originSessionId: 42bf1a19-7f07-405b-a1a4-451da110e784
---

The monorepo's `overmind.md` rule says to restart a service with `overmind stop <name>` then `overmind start <name> -D`. **This does not work** — when the overmind daemon is already running, `overmind start <name> -D` tries to spawn a *new* daemon and errors with "it looks like Overmind is already running".

**What actually works:** after `overmind stop <name>` the process shows as `dead`; bring it back with `overmind restart <name>`. Since the proc is already dead, `restart` starts it exactly once (no duplicate-process risk — the rule's concern about duplicates only applies to restarting a *running* proc).

**Why:** picked up restarting `bola-api` after an `.env` change (2026-06-22). Air rebuild takes ~10s; verify with `curl http://localhost:8097/system/readiness`.

**How to apply:** to restart a single overmind-managed service, just use `overmind restart <name>` (don't follow the stop+start-D sequence from overmind.md). Note: `overmind echo` streams logs forever and will hang a Bash call — avoid it for one-shot log checks.
