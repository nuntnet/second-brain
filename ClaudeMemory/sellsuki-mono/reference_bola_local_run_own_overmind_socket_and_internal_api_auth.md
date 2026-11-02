---
name: bola-local-run-own-overmind-socket
description: How to run bola-backend locally (not in the default stack) and the machine-auth/DB facts needed to verify BOLA internal endpoints
metadata:
  node_type: memory
  type: reference
  originSessionId: 02887868-ba40-4bca-a6c9-fdfb2c10d3d7
  modified: 2026-11-02T08:55:44.816Z
---

- bola-api is NOT started by default. Start only it: `rm -f .overmind-bola-api.sock; overmind start -f Procfile.bola -l bola-api -s .overmind-bola-api.sock -D` (port **8097**, air builds ~1-2 min; poll /system/liveliness in a bounded loop, `sleep` chains are blocked). The default `overmind status` says "not running" even when this one is up.
- Get a branch onto it with `scripts/local-stack.sh add backend/bola-backend <branch>` (path form, not `bola-backend`).
- **After `add`, check air actually rebuilt**: compare `stat -f %m backend/bola-backend/tmp/main` with the merge commit time. On 2026-11-02 air's watcher was dead (binary from 3 days earlier, liveliness still 200, so it looked fine). Fix: `overmind quit -s .overmind-bola-api.sock`, wait for :8097 to free, then the start line above; confirm a build with the default GOCACHE works first or air leaves :8097 down. A stack branch that moved after its merge shows as `!! DROPPED` in `local-stack.sh status`; `add` again merges the new commits.
- Machine auth for BOLA is `X-System-Token` = `SYSTEM_ADMIN_TOKEN` in backend/bola-backend/.env (BOLA-264). Read it inside a bash command with `sed -n 's/^SYSTEM_ADMIN_TOKEN=//p' ... | tr -d '"'`; never print it. chat-core's old client sends `X-Service-Token`, which BOLA does not accept.
- Local DB: `docker exec sellsuki_mono-postgres-1 psql -U postgres -d bola`. Existing workspaces mostly have empty `company_id`; seed your own `b<card>-` prefixed company/workspace/OA/line_group_chats rows (NOT NULL: workspaces id/name/slug; line_oas id/workspace_id/name/channel_id/channel_secret/channel_access_token).
- Go build/test in worktrees needs unsandboxed Bash (go cache). Worktree for BOLA goes in `backend/bola-backend/.worktrees/<name>`.
- Related: [[reference_sandbox_blocks_go_build_cache_and_bsd_sed_noop_mutation]]
