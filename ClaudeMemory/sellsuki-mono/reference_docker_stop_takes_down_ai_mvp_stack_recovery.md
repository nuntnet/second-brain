---
name: reference_docker_stop_takes_down_ai_mvp_stack_recovery
description: "When Docker Desktop stops, the whole AI MVP local stack goes with it (overmind ai-mvp socket gone, Kafka exits 143); recovery order and who runs what"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7a3a9e06-b9ae-4a73-b6de-ff58721db369
  modified: 2026-11-04T02:20:20.907Z
---

Seen 2026-11-04 ~08:10: Docker Desktop stopped by itself (only `com.docker.vmnetd` left, disk fine). Effects: local Postgres/Redis/Kafka down, DB-backed Go tests **skip silently** (a run reported "15 ok" while one real-Postgres test failed on connection refused), and the overmind `.overmind-ai-mvp.sock` session was gone with every service port closed.

Recovery, in order:
1. nunt starts Docker Desktop (the sandbox blocks `open`).
2. `docker compose up` containers come back, but `sellsuki_mono-kafka-1` stays **Exited (143)**: `docker start sellsuki_mono-kafka-1` (agent may do this; it restores the prior stack).
3. nunt runs `scripts/ai-mvp-restart.sh` in a terminal (it reads `.env.ai-mvp` keys, which agents must not read; it also starts `sellsuki-rag-postgres`). If it starts nothing, it probably refused on missing keys: ask nunt to run it in the app's own terminal tab so `read_terminal` can see the output.
4. Check ports 4455 8099 8100 8101 8001 5191 9999 9998 8087 8085 and `/system/readiness` on 8099/8101 before trusting any local result.

`scripts/local-stack.sh add` ignores untracked files (guard uses `--untracked-files=no`) even though `status` prints them as `dirty=1`; untracked files in a checkout do not block an add. After adding a console branch that changed the lockfile, run `pnpm install --offline --frozen-lockfile` in the main checkout before Vite starts.

Related: [[project_overmind_restart_quirk]], [[reference_docker_desktop_half_dead_after_disk_full]], [[reference_sandbox_blocks_localhost_so_db_tests_skip_silently]].
