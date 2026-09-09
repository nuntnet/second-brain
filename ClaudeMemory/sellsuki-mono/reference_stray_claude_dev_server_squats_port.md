---
name: reference-stray-claude-dev-server-squats-port
description: "A dev server an earlier Claude session started with `go run`/`vite` can hold a port for days — `overmind status` reports the proc dead while the port answers, and `make clean-procs` does NOT catch it; symptom is edits/pull/restart having no effect"
metadata:
  node_type: memory
  type: reference
---

Observed 2026-08-14 on `bola-backend` (port 8097) and `bola-frontend` (5184). Both
had been started by a **Claude Code session on 2026-08-12** and were still running
two days later:

```
83118 /Applications/Claude.app/.../disclaimer /bin/bash -c   ← parent is Claude.app
83120 go run ./cmd/bola_server
83167 .../go-build.../exe/bola_server                        ← holds 8097
75951 node .../scratchpad/bolafe-local/.../vite --port 5184  ← holds 5184
```

**Why it is so hard to see:**

- `OVERMIND_SOCKET=.overmind-bola.sock overmind status` says the proc is **`dead`**,
  yet `curl localhost:8097` answers — so "the service is running" and "overmind is
  running it" look like the same thing and are not.
- `make clean-procs` (`scripts/kill-zombies.sh`) **misses it**: it hunts zombies,
  orphaned `tmp/main` whose air parent died, and stale air. This process has a live
  parent chain and is none of those.
- `go run` compiles **once at start and never rebuilds**, so `git pull` +
  `overmind restart` change nothing, forever. The frontend equivalent served an old
  worktree on a detached HEAD.

**Diagnosis that actually works** — a 404 body tells you which layer:

| response | meaning |
|---|---|
| `401 {"error_code":"missing_credential"}` | route EXISTS, auth rejected it |
| `404 Cannot POST /v1/...` (Fiber's own text, no `error_code`) | router has no such route → old binary |

Then confirm what is really serving:

```bash
ps aux | grep -E "go run|tmp/main|vite --mode" | grep -v grep
strings <repo>/tmp/main | grep <a-string-your-fix-added>   # is the fix IN the binary
```

**Fix:** `kill <child> <parent>` (child first — `go run` does not release its child),
then `overmind restart <name>`. The second step is required: after the squatter dies
air's own process is alive but **never bound**, and it does not retry on its own.

This is exactly what `.claude/rules/overmind.md` rule 1 ("NEVER start services
directly — no `go run`, `air`, or executing binaries") exists to prevent — and the
violator was a Claude session. See [[project_overmind_restart_quirk]] and
[[reference_local_bola_own_overmind_socket]].
