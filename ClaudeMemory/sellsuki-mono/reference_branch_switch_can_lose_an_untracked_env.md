---
name: reference_branch_switch_can_lose_an_untracked_env
description: messaging-backend's gitignored .env vanished during a git checkout between branches and the service died with "dial tcp: lookup port=5432" — recover it from the commit an earlier session made on feature/mvp-media-and-secret-durability
metadata:
  type: reference
---

Switching `backend/sellsuki-messaging-backend` from a feature branch to `main`
left the repo with **no `.env`**, even though `.env` is in `.gitignore` and
`git ls-files` does not know it. The service then died at boot with a message
that points nowhere near the cause:

```
Failed loading .env file: open .env: no such file or directory
FATAL  Error establishing connection to postgresql
       {"error":"dial tcp: lookup port=5432: no such host"}
```

`lookup port=5432` is the tell: with no `.env`, `POSTGRES_HOST` is empty and
the DSN collapses to something that parses but cannot resolve. It reads like a
network or Postgres problem and is neither.

Worse, overmind reports the process as **running** while nothing listens —
air restarts the binary, the binary exits 1, and `tmp/build-errors.log` fills
with a bare `exit status 1` that says nothing about why. The real message is
only in the tmux pane:

```bash
S=$(ls -t /private/tmp/tmux-501/ | head -1)
tmux -S /private/tmp/tmux-501/$S capture-pane -p -t sellsuki-mono:messaging -S -40
```

## Recovery

An earlier session committed a snapshot of it — `1482535 chore(local): เก็บ
.env ที่แก้ไว้ในเครื่องก่อนย้ายไป develop`, on branch
`feature/mvp-media-and-secret-durability`:

```bash
git show 1482535:.env > .env
```

26 keys, including `POSTGRES_*`, `VAULT_*`, `CHAT_SERVICE_TOKEN` and
`CHAT_CORE_*`. Re-append anything added since — the shared FB app pair
(`CHAT_FB_SHARED_APP_ID`, `CHAT_FB_SHARED_APP_OWNER_COMPANY_ID`) is also set
in `Procfile.ai-mvp`, which stays authoritative for the local-run topology.

## Before switching a branch in a repo with a live local stack

Copy the `.env` somewhere first. It is untracked by design, so nothing in git
will bring it back if it disappears, and the failure it causes names the wrong
subsystem.

Related: [[reference_moving_a_branch_under_a_running_dev_server]],
[[reference_overmind_dead_processes_are_a_wedged_session]].
