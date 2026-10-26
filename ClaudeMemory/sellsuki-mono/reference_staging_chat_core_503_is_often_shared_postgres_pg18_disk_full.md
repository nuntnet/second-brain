---
name: reference_staging_chat_core_503_is_often_shared_postgres_pg18_disk_full
description: "2026-10-26 staging chat-core answered 503 on every path: pods Running 0/1 (readiness /system/readiness 503, 0 restarts), logs \"connection refused\" to postgresql-pg18.datastore; the DB pod was CrashLoopBackOff with \"No space left on device\" writing pg_wal - disk full, SRE/DBA job, not an app deploy"
metadata:
  node_type: memory
  type: reference
  originSessionId: 28036f73-9028-4b3a-98a8-7ab8e1f2c879
  modified: 2026-10-26T07:18:50.492Z
---

How it was found (all read-only, from the user's terminal because Go CLIs cannot verify TLS in the sandbox, see [[reference_sandbox_blocks_go_cli_tls_and_ssh]]):

1. `curl` the staging host outside the sandbox (inside it `http=000` is the sandbox allow-list, not an outage): chat-core `/`, `/health`, `/healthz` all 503, console 200 (a SPA answers 200 to any path, proves nothing).
2. Rule out your own merges first: the chat-core `deploy_staging_th_arm` job on main is `manual`; the last real deploy was before the merges. A merge to main does not deploy by itself.
3. `kubectl -n sellsuki get pods | grep chat-core`: `0/1 Running`, 0 restarts = readiness failing, not crashing. `describe pod`: `Readiness probe failed ... statuscode: 503` (and timeouts).
4. `kubectl -n sellsuki logs <pod> --tail=40`: `failed to connect to user=chat_core database=chat_core: 172.20.x.x:5432 (postgresql-pg18.datastore): connection refused`.
5. `kubectl -n datastore get pods -o wide | grep pg18`: `postgresql-pg18-0 1/2 CrashLoopBackOff`. `kubectl -n datastore logs postgresql-pg18-0 --all-containers --previous --tail=25`: `FATAL: could not write to file "pg_wal/xlogtemp.N": No space left on device`, "database system was interrupted while in recovery".

The readiness endpoint of chat-core fans out to its dependencies, so one dead dependency (here the shared Postgres) makes every pod unready and the cron jobs that call it end in `Error`.

Do not delete WAL files or the pod to "fix" it: the volume needs expanding or freeing by whoever owns `datastore`. Report it with the log lines above. The shared pg18 serves other staging services too, so expect more than chat-core to be down.
