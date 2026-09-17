---
name: reference_overmind_dead_processes_are_a_wedged_session
description: When overmind shows a process "dead" and it restarts straight back to dead while the same command runs fine by hand, the session is wedged — quit it, remove the socket and start it again; also never run a service by hand in a repo overmind is watching
metadata:
  type: reference
---

`overmind status` showed chat-core and messaging **dead** on
`.overmind-ai-mvp.sock` while ai-agent and rag-core in the same session ran
fine. `overmind restart <name>` assigned a new PID that died instantly.
Running each service's exact Procfile line by hand worked perfectly.

It was not the services. The **session was wedged**. The fix:

```bash
OVERMIND_SOCKET=$PWD/.overmind-ai-mvp.sock overmind quit
rm -f .overmind-ai-mvp.sock
make dev-ai-mvp          # all 8 processes came up
```

Note `overmind start <name>` does NOT start one process in a running session —
it tries to start a whole new session and answers "it looks like Overmind is
already running". For a live session the per-process command is `restart`,
despite what the repo's overmind rule suggests.

Contributing state: **67 stale tmux sockets** under `/private/tmp/tmux-501/`,
every one holding a session named `sellsuki-mono` (the name comes from the
directory, so every overmind session on this repo collides on it).

## Two traps worth more than the fix

**A service started by hand does not get the Procfile's env.** `Procfile.ai-mvp`
sets `ORY_KRATOS_PUBLIC_URL` and `ROLE_PERMISSION_SERVICE_GRPC_URL` on the
chat-core line, and the file says outright that it, not `.env`, is
authoritative for local topology (godotenv does not override an already-set
variable). Running chat-core by hand produced a stack where every
authenticated `/v1` request answered 503 — and that got misdiagnosed as a
missing repo config for hours. Check the Procfile line before concluding a key
is missing anywhere.

**Two `air` processes in one directory fight.** `.air.toml` has
`clean_on_exit = true`, so running a service by hand in a repo overmind is
already watching deletes `tmp/` on exit — including the binary the
overmind-managed air is running. Symptom: overmind says "running", nothing is
listening. `overmind restart <name>` recovers it.
