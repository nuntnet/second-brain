---
name: reference-local-bola-own-overmind-socket
description: "Local bola runs on its own overmind socket (.overmind-bola.sock + Procfile.bola), separate from the main .overmind.sock session — 502 on bola.sellsuki.local usually just means it isn't started"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T13:40:41.507Z
---

The main `.overmind.sock` session in this workspace usually holds a **stack subset** (observed 2026-08-11: only the OC2Plus set — oc2plus-api, kratos-ui, role-perm, address, ccs, sellsuki-pay, consent, web-oc2plus), not everything in `Procfile`. bola is not in it, and there is **no `make dev-bola` target** — start it on its own socket like the shipmunk / ai-mvp / multipod stacks do:

```bash
rm -f .overmind-bola.sock   # stale socket from a dead session is common
OVERMIND_SOCKET=$PWD/.overmind-bola.sock overmind start -f Procfile.bola -N -D
```

Then `OVERMIND_SOCKET=$PWD/.overmind-bola.sock overmind status`. Backend 8097, frontend 5184; Caddy fronts both at `bola.sellsuki.local` (`/v1/*` etc → 8097, rest → SPA) plus `bola-api.sellsuki.local`.

**Two overmind gotchas learned the hard way (2026-08-11):**
- To start ONE process from an existing Procfile without launching the rest: `overmind start -f Procfile.frontend -l web-company -N -D` on its own socket. Do NOT hand-write a Procfile in `/tmp` — overmind resolves commands relative to the **Procfile's** directory, so a Procfile outside the repo makes every `cd frontend/...` fail and the whole session exits instantly (socket vanishes, no error shown).
- Inside an already-running session, `overmind start <name> -D` fails with "it looks like Overmind is already running" — `-D` means "daemonize a new instance". Use `overmind restart <name>` to bring one process back (this contradicts the repo rule that says stop-then-start; restart is what actually works here).

**Why:** a separate socket means starting bola never disturbs whatever the main session is running — important because [[feedback-parallel-sessions-git-safety]] applies to processes too, not just git.

**How to apply:** distinguish the two local failure modes before touching anything —
- **502 Bad Gateway** on one host = Caddy healthy, upstream not listening → the service isn't started (check `lsof -nP -iTCP:8097 -sTCP:LISTEN`).
- **connection refused across ALL `*.sellsuki.local`** = the Caddy container itself → see [[reference-caddy-host-networking-gotcha]].
