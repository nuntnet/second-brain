---
name: project_overmind_restart_quirk
description: "overmind — `overmind start <name> -D` does NOT work as CLAUDE.md claims; use `overmind restart <name>` for a dead proc"
metadata: 
  node_type: memory
  type: project
  originSessionId: 42bf1a19-7f07-405b-a1a4-451da110e784
  modified: 2026-08-26T04:22:41.015Z
---

The monorepo's `overmind.md` rule says to restart a service with `overmind stop <name>` then `overmind start <name> -D`. **This does not work** — when the overmind daemon is already running, `overmind start <name> -D` tries to spawn a *new* daemon and errors with "it looks like Overmind is already running".

**What actually works:** after `overmind stop <name>` the process shows as `dead`; bring it back with `overmind restart <name>`. Since the proc is already dead, `restart` starts it exactly once (no duplicate-process risk — the rule's concern about duplicates only applies to restarting a *running* proc).

**Why:** picked up restarting `bola-api` after an `.env` change (2026-06-22). Air rebuild takes ~10s; verify with `curl http://localhost:8097/system/readiness`.

**How to apply:** to restart a single overmind-managed service, just use `overmind restart <name>` (don't follow the stop+start-D sequence from overmind.md). Note: `overmind echo` streams logs forever and will hang a Bash call — avoid it for one-shot log checks.

**⚠️ Update (2026-08-14) — `overmind restart <name>` is NOT always safe, even on a proc `overmind status` shows as "running".** Ran `overmind restart ccs` mid-session and it killed the *entire* default-socket session (`.overmind.sock` disappeared, every other proc in that session went down too), not just `ccs`. Root cause unclear (possibly two competing sessions touching the same default socket), but the safe recovery is: `rm -f .overmind.sock` (or use `docker`-style force-recreate for Caddy specifically), then `overmind start -l <full,list,of,services> -D` naming *every* service you need back, not just the one that died — `overmind` doesn't support adding one proc to an already-running session. Also: `overmind status`/`remain-on-exit tmux` can both lie that a proc is "running" when the actual binary crashed (fail-fast health check) — verify with a real port check (`lsof -iTCP:<port> -sTCP:LISTEN`) or `curl` the service, not the overmind status table alone.

**⚠️ Procfile name collision:** the Procfile process named `ccs` runs `backend/sellsuki-central-control-backend` (company/provider/user management), **not** `backend/central-configuration-system` (the config-store service, unmanaged by this Procfile entry — if running locally it's likely a separate orphaned/manually-started process on :8085). Don't assume "ccs" means central-configuration-system in this repo's tooling.
