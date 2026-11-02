---
name: reference_enfile_outage_corrupts_go_build_cache_and_kills_air_watchers
description: "After the 2026-11-02 file-table (ENFILE) outage the default Go build cache was corrupt (std packages read as 'not in std') so every air rebuild failed silently and services kept old binaries; air watchers also died. Fix: go clean -cache, then overmind stop + overmind restart -s <socket> <name>"
metadata:
  node_type: memory
  type: reference
  originSessionId: 412dc360-5d42-492b-8f38-95d6097c2b92
  modified: 2026-11-02T04:47:25.450Z
---

Seen 2026-11-02 (AI-385) after the machine's file table filled (see [[reference_codegraph_watchers_exhaust_system_fds]]).

- Symptom: `go build`/`go test` with the default `GOCACHE` (~/Library/Caches/go-build) fail with `package io/fs is not in std` / `package os/exec is not in std` although the toolchain dir is intact; a fresh `GOCACHE=<scratch>/gocache` builds fine.
- Consequence nobody sees: air's rebuild fails (`tmp/build-errors.log` = `exit status 1`) and the service keeps serving the OLD `tmp/main` (chat-core was on an Oct 30 build). Check `ls -la <svc>/tmp/main` before trusting any live result.
- Air's file watcher was dead too: touching a .go file triggered nothing.

**How to apply:** `go clean -cache` (a regenerable cache). Then for a stuck AI-stack process: `overmind stop -s .overmind-ai-mvp.sock <name>` and bring it back with `overmind restart -s .overmind-ai-mvp.sock <name>` — `overmind start -s … -D <name>` tries to launch a second manager and fails with "Overmind is already running", leaving the process dead. Related: [[reference_overmind_restart_dead_process_ai_mvp_socket]].
