---
name: reference_docker_desktop_half_dead_after_disk_full
description: "After the disk filled, Docker Desktop stayed half-dead (backend alive, engine gone, no ~/.docker/run/docker.sock); `docker desktop start` says \"already running\" — needs killall -9 com.docker.backend; then Kafka/BOLA/backoffice must be revived"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7fb71c5f-6e71-4687-b858-9caa035598f1
  modified: 2026-10-10T09:34:27.742Z
---

Seen 2026-10-10. The disk filled (155 MB free). Docker Desktop's engine and UI shut down, but the `com.docker.backend` processes (started days earlier) never exited. Results:
- Reopening Docker.app does nothing.
- `docker desktop start` answers "already running".
- `docker desktop restart` fails with "processes still running … context deadline exceeded".
- `~/.docker/run/` is empty.

To check the state, read the end of `~/Library/Containers/com.docker.docker/Data/log/host/com.docker.backend.log`: the last lines are "shutting down vital services".

**Fix:** the user runs `killall -9 com.docker.backend`, then `docker desktop start --timeout 300`. Free disk first. `go clean -cache` freed 21 GB.

**After Docker comes back, the stack does not recover by itself:**
- Containers that died with exit 255 stay down. Kafka is one of them: `docker start sellsuki_mono-kafka-1`.
- Services whose binary died at boot (backoffice-api: Kafka dial fatal; bola-api: Postgres dial fatal) leave `air` running with no child process. Re-trigger air by rewriting a watched file with identical content. Do not kill or restart anything.
- Caddy needs `docker compose up -d --force-recreate caddy` (see [[reference_caddy_host_networking_gotcha]]).

To find a service's log: `ps -ax | grep "tmux -C -L overmind" | grep " <name> "` gives the `-L` socket; then run `tmux -L <sock> capture-pane -p -J -S -200 -t <name>`.
