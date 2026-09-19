---
name: it-exists-but-cannot-reach-the-cluster
description: "Four times in one night on rag-core, the thing needed already existed and simply could not reach the environment — and every time the layer that reported success was not the layer that failed. Check the artifact in the cluster, never the job that produced it."
metadata:
  type: reference
---

2026-09-20, unblocking "KB into a chat workspace" on staging. Four separate
blockers, one shape:

| # | What existed | Why it had no effect | What reported success |
|---|---|---|---|
| 1 | a newer image, built fine | a boot guard demanded a var nobody had set | `kubectl get deploy` 1/1 — Helm `--atomic` rolled each deploy back |
| 2 | a CronJob, deployed fine | `CRON_IMAGE_NAME` was the SERVICE name, not the image repo | the deploy job — Helm installs a CronJob without ever pulling |
| 3 | seven Alembic migrations, merged | nothing runs them on a real environment (only a storage-test job, against a throwaway DB) | CI, which ran them green against that throwaway |
| 4 | a Milvus collection provisioner | it lived in `scripts/`, which the Dockerfile does not `COPY` | nothing — it simply had never run anywhere but a laptop |

## The rule

**Verify the artifact in the environment, not the job that produced it.**

- routes → `kubectl port-forward` + `curl /openapi.json`, compare to what the
  repo's router assembly mounts
- schema → `kubectl exec deploy/<x> -- alembic current` (or the ORM's
  equivalent), compare to the newest migration file
- a CronJob → **wait one schedule interval and read the pod.** The deploy job
  succeeding proves nothing
- a Milvus/vector collection → ask the running pod, using its own credentials,
  so you never handle the token
- an env var → parse the Deployment's rendered env, not the values file

## The second-order lesson

#3 and #4 were invisible because they hid behind each other: nothing read those
tables because the worker that reads them had never been deployed, and nothing
noticed the missing collection because nothing got as far as writing to it.
**A gap nobody exercises is a gap nobody reports.** When several pieces of a
path have never run, expect one blocker per piece — fixing the first only buys
the right to see the second.

Related: [[reference_boot_guard_pins_an_old_image_silently]] ·
[[reference_helm_installs_cronjob_without_pulling]] ·
[[reference_migration_files_are_not_applied_schema]] ·
[[reference_ci_history_and_dns_are_not_deploy_status]]
