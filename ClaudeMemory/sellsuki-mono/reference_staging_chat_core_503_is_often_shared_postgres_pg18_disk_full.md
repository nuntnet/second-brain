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

**Fix applied 2026-10-26 15:09 (SRE said go):** `kubectl -n datastore patch pvc data-postgresql-pg18-0 -p '{"spec":{"resources":{"requests":{"storage":"300Gi"}}}}'` (200Gi -> 300Gi; storage class `datastore` is ebs.csi with allowVolumeExpansion; EBS allows one modification per ~6 h, so choose generously). The controller resized in ~14 s, kubelet expanded the filesystem online (`FileSystemResizeSuccessful`), the crash-looping pod came back on its own at the next backoff (~2 min, no pod deletion), Postgres finished recovery ("ready to accept connections" 08:11 UTC), chat-core pods turned Ready and `/system/readiness` answered 200. After: `/dev/nvme4n1 295G, 197G used, 99G avail (67%)`. The 197 GiB was NOT freed, so the root cause (what grew 96 -> 196 GiB from 19 Oct; kubelet metric stops 24 Oct 14:00; no pg_database_size_bytes in Mimir) is still open: check per-database and per-table size, pg_replication_slots, pg_stat_archiver. The postgres container is named `postgresql` (sidecar `metrics`). Mimir is reachable by port-forward to `svc/mimir-nginx` (80), path `/prometheus/api/v1/query_range`.

**Root cause found 2026-10-26 15:25 (read-only psql through `kubectl exec ... -c postgresql`, session forced read-only with `PGOPTIONS='-c default_transaction_read_only=on'`; run it from a script file, the terminal tool refuses `exec ... < file` on the command line):** `pg_wal` was **157 GB in 10,064 files** of the ~197 GiB used. It was held by two INACTIVE logical replication slots of Debezium for `develop-ms-service-pis` (`debezium_develop_ms_service_pis` and `..._history`, `active=f`, restart_lsn 72/386E2220, 157 GB retained each), with `max_slot_wal_keep_size = -1` (unlimited), `wal_level=logical`, `archive_mode=off` (so not an archiver problem). Two more inactive slots for `staging-ms-service-pis` retain 917 MB and are growing. Actual data is small: staging_ory_keto 18 GB (keto_relation_tuples, 13 GB of it indexes), staging_ory_kratos 6 GB, bola_staging 2.3 GB, chat_core 26 MB. So the full disk was NOT caused by chat-core or the AI chat work. The fix that frees the space is reviving the PIS Debezium connectors or dropping the dead slots (`pg_drop_replication_slot`, owner's decision, irreversible for the consumer) plus setting `max_slot_wal_keep_size` and a disk/slot-lag alert; expanding the PVC only bought time.
