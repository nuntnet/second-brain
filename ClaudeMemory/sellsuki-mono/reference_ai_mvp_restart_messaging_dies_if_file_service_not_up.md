---
name: reference_ai_mvp_restart_messaging_dies_if_file_service_not_up
description: "After scripts/ai-mvp-restart.sh, messaging (:8100) can FATAL at boot because file-service (:8087) isn't up yet; air won't restart it — `overmind restart -s .overmind-ai-mvp.sock messaging` once 8087 answers"
metadata:
  node_type: memory
  type: reference
  originSessionId: 8b69c115-30aa-4ff4-b9c5-86fafae17c77
  modified: 2026-11-09T05:57:51.818Z
---

Seen 2026-11-03 on the first `scripts/ai-mvp-restart.sh` run (round 1 of the AI chat MVP). The restart script reported
`NOT UP after 4 minutes: 8100 50057`. The messaging tmux pane showed
`FATAL generics_server/helper.go:579 Error health check fail before start (chat module) … dial tcp [::1]:8087: connect: connection refused`.
That is a startup race: messaging's chat module health-checks file-service before file-service has finished building.

- air does not restart a binary that exited. The process stays "running" in `overmind status`, but nothing listens on 8100.
- Fix: once `nc -z localhost 8087` succeeds, run `overmind restart -s .overmind-ai-mvp.sock messaging`. That is the allowed
  exception for a dead process, see [[reference_overmind_restart_dead_process_ai_mvp_socket]]. The boot line to look for is
  `secret store: encrypted postgres (chat.secret_store), active key id k1, vault dual read true`.
- 50057 (file-service gRPC) never comes up on this stack. Nothing in the AI stack uses it, so ignore it.
- To read a pane without attaching, find the socket with `ls /private/tmp/tmux-501/`, then for each one
  `tmux -L <sock> list-windows -a -F '#{session_name}:#{window_name}'` to find `:messaging`, and
  `tmux -L <sock> capture-pane -p -t sellsuki-mono:messaging -S -80`. Note that `ps -ef | grep 'tmux -C'` does NOT list the ai-mvp socket.
