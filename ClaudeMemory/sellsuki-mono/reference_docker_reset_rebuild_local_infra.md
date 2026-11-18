---
name: reference-docker-reset-rebuild-local-infra
description: "After Docker Desktop resets (2026-11-18, resource-limit change) all containers, images and NAMED volumes are gone; data/ bind mounts survive. Rebuild needs 3 fixes: postgis bullseye apt archive, minio image withdrawn, rag-core volume lost"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-18T14:52:08.586Z
---

2026-11-18 21:00: Docker Desktop's disk image was reset. `docker ps -a` empty, no images, named volumes gone.

- Survives: everything bind-mounted under `data/` (main Postgres 5.3 GB, minio, redis, mongo...).
- Lost: `rag-core_sellsuki_rag_pgdata` (named volume): recreate with `cd backend/rag-core && docker compose -f docker-compose.local.yml up -d postgres && uv run alembic -c alembic.ini upgrade head`. Local KB entries are gone.
- `make infra` fails on three things, all fixed in the working tree on 2026-11-18 (uncommitted at first):
  1. `docker/postgres/Dockerfile`: postgis:16-3.4 is bullseye; `apt.postgresql.org` moved bullseye-pgdg to `apt-archive.postgresql.org`, and debian-security bullseye 404s on python3.9 → sed the pgdg list + drop debian-security.
  2. `minio/minio` is withdrawn from Docker Hub (pull access denied; quay.io 401) → `cgr.dev/chainguard/minio:latest`, reads the same `data/minio`.
  3. The postgis image is amd64-only → runs emulated on this ARM Mac (a real CPU cost).
- After infra returns, `scripts/ai-mvp-restart.sh`; it can leave overmind running with its socket file deleted → kill that overmind by PID (SIGTERM) and rerun.
- file-service checkout sat 6 behind origin/main where gRPC (:50057) was still commented out → `git merge --ff-only origin/main`.

Related: [[reference-local-duplicate-overmind-chat-core-and-procfile-reload]].
