---
name: disk-full-kills-docker-desktop-go-cache-is-the-first-thing-to-delete
description: 2026-10-25 the Mac data volume hit 100% (go build caches 19 GB + Docker.raw ~100 GB) → every Bash call refused (ENOSPC), Docker engine shut down, chat-core crashed on 'connection refused'. Recovery order and what is safe to delete
metadata:
  node_type: memory
  type: reference
---

What happened (2026-10-25 ~11:11): `/System/Volumes/Data` 460 GB, 1.6 GB free. Biggest items: `Docker.raw` (~100 GB real, apparent 494 GB, never shrinks), two Go build caches I had created with `GOCACHE=$TMPDIR/gocache` and the session scratchpad (10 GB + 9 GB). Bash tool refused to start ("disk is full, ENOSPC"); Docker Desktop's engine did `POST /shutdown` and its UI exited; `com.docker.backend` stayed wedged (ignored SIGTERM) so the app could not be reopened.

Recovery that worked: (1) delete only what the agent created — `rm -rf $TMPDIR/gocache` and `<session scratchpad>/gocache` (they regrow; Go rebuilds) → 20 GB free; (2) `kill -TERM` then `kill -KILL` the stale `com.docker.backend` / `com.docker.build` / `docker-agent` processes (NOT `com.docker.vmnetd`, NOT the `com.apple.Virtualization.VirtualMachine` XPC unless `lsof` shows it holds Docker.raw), (3) the user reopens Docker.app (Launch Services is blocked from the sandbox), (4) `docker start sellsuki_mono-kafka-1` (it exits 255 with the VM; postgres, redis, vault, kratos, keto came back on their own), (5) run `MIGRATE_HEADLESS=true DATABASE_CHAT_CORE_POSTGRESQL_URI=postgres://postgres:postgres@localhost:5432/chat_core?sslmode=disable go run ./cmd/migration` from backend/sellsuki-chat-core (a migration run while Postgres is wedged just hangs: check `docker ps` first), (6) chat-core had FATAL-exited at boot (`failed to initialize database`) and air does not restart a crashed binary: rewrite one tracked .go file with identical bytes (`touch` does not trigger fsnotify WRITE) and air rebuilds.

Rules to keep: use ONE shared `GOCACHE=/tmp/claude-501/gocache` for every agent and delete it when the work is done; never let workflow agents create their own caches; check `df -h /System/Volumes/Data` before launching a multi-agent workflow; do not delete Docker data or user folders without asking (`docker image prune` / `docker builder prune` are the safe Docker ones, never `--volumes`).
Reading tmux logs of the overmind stack without attaching: `tmux -L <socket from ps -ef|grep 'tmux -C'> capture-pane -p -t sellsuki-mono:chat-core -S -40`.
Monitors that poll a port can expire with "still not listening" while the service came up a minute later: always re-check directly before reporting.
