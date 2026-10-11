---
name: reference-overmind-is-many-instances-one-socket-per-proc
description: "The local stack is ~20 separate overmind instances (one per proc or Procfile, each with its own -s socket), not one; the default .overmind.sock shows one proc; restart a proc through ITS socket; I killed web-oc2plus's Vite by pid on 2026-10-11 and its overmind died with it"
metadata:
  node_type: memory
  type: reference
  originSessionId: 5077123b-d938-4a36-b54b-335561e7b9a9
  modified: 2026-10-11T02:04:33.294Z
---

Observed 2026-10-11 09:00: `ps -o command -A | grep "overmind start"` listed ~20
instances, e.g. `overmind start -f Procfile -l kratos-ui -N -D`, `… -f Procfile.oc2api
-N -D`, `… -f Procfile.bola -l web-bola -s .overmind-bola-web.sock -N -D`, several
`Procfile.kb-rag` ones with `.overmind-kb-*.sock`. `overmind status` against the
default `.overmind.sock` showed only `web-ai-chat-admin`. Each instance owns a tmux
server `overmind-sellsuki-mono-<id>` under `$TMPDIR`; `tmux -L <that> capture-pane -p
-t sellsuki-mono:0 -S -400` reads a proc's log without touching it.

So "one stack per machine" (`shipping.md` §8) is one *set* of instances; the
`overmind.md` rules still hold per instance. To stop/restart a proc:
`OVERMIND_SOCKET=.overmind-<x>.sock overmind stop <proc>` then `… start`, or start a
fresh instance for it the house way: `overmind start -f <Procfile> -l <proc> -s
.overmind-<proc>.sock -N -D`.

**What I did wrong:** the user asked to "restart web-oc2plus"; I saw a `bun run
start:dev` with no entry in the default `overmind status` and `kill`ed the bun and
node pids. The parent `sh` was an overmind tmux pane (`overmind-sellsuki-mono-SEO06…`)
and that instance exited when its only proc died. Recovery: a new instance via
`overmind start -f Procfile.oc2plus-fe -l web-oc2plus -s .overmind-oc2plus-fe.sock -N -D`
(single listener on 5176 confirmed). `scripts/doctor.sh` now lists every instance.

**How to apply:** before stopping anything, `ps -o pid,ppid,command -p <pid>` up two
levels; a parent path under `$TMPDIR/overmind-*` means "use that instance's socket".
Never signal a service pid. Related: [[feedback-generator-must-not-wipe-shared-dirs]].
