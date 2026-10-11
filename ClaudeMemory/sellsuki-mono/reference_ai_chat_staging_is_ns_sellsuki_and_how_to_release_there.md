---
name: reference_ai_chat_staging_is_ns_sellsuki_and_how_to_release_there
description: "AI chat stack (chat-core, rag-core-python, ai-agent StatefulSet, crons, admin FE) runs in ns `sellsuki` on staging-th = what CI calls staging; dev-th (`sellsuki-dev`) has none of it. How to release there by hand: migration Job manifests, manual deploy jobs, port-forward checks; CI runner capacity failures and how to retry"
metadata:
  type: reference
---

**Where it runs (checked 2026-10-07):** cluster staging-th (context `teleport.internal.staging-th…`), namespace **`sellsuki`**
(`.variables_export_staging_arm: KUBE_NAMESPACE=sellsuki`): `sellsuki-chat-core` Deployment, `rag-core-python` Deployment
(the one chat-core calls; `rag-core` Go deployment is legacy), `sellsuki-ai-agent` **StatefulSet** (pod `-0`, PVC), cronjobs
`rag-core-python-ingest-cron`, `rag-core-python-sheet-sync-cron`, `sellsuki-chat-core-sla-sweep-cron`, `-trace-retention-cron`;
FE at `ai-chat-admin.staging.sellsuki.com` → `chat-core.staging-th.sellsuki.com`. Secrets: `sellsuki-chat-core-common-secret`
(8 keys incl. DB URI, SERVICE_TOKEN, REDIS_URL, KAFKA_BROKERS), `sellsuki-rag-core-secret` (17 keys incl. POSTGRES_DSN, MILVUS_*,
OPENROUTER_API_KEY), `rag-core-knowledge-pipeline-secret`, `sellsuki-ai-agent-*-secret`.
**dev-th (`sellsuki-dev`) has no AI chat stack** — only `sellsuki-chat-core-client-secret` (messaging-backend's). Building it is
the "Option B" in `docs/ai-chat-dev-env-plan.md` (SRE work). Memory `reference_ai_chat_has_no_staging_deployment` is stale for
chat-core (deployed since ~2026-09-19).

**Release by hand (Option A, done for rag-core on 2026-10-07):**
1. Merge to `main` → pipeline builds image `registry.fountain.sellsuki.com/service/<repo>:<8-char sha>` (read the tag from the
   build job trace); `deploy_staging_th_arm` and cron deploys are **manual** jobs → `glab api -X POST projects/:id/jobs/<id>/play`.
2. Migration first: copy `deployment/job-migration-staging.yaml`, replace the job name suffix and the image tag, `kubectl apply`,
   wait for `.status.succeeded=1`, read `kubectl logs job/<name>` (alembic prints `Running upgrade A -> B`). chat-core's job runs
   `/app/migrations` with `MIGRATE_HEADLESS=true` and the DB URI from the secret. Record it in `docs/release/environments.yaml`
   under `staging:` (the file's staging section is OC2Plus-flavoured; a comment says the AI stack is ns `sellsuki`).
3. Verify through `kubectl -n sellsuki port-forward svc/rag-core-python-svc 18001:80` → `/readyz`, and a new route answering 401
   without a token (not 404).
**CI trap:** MR jobs die `Unschedulable: 0/12 nodes … Insufficient cpu/memory` (runner pods in ns `share`); they show as failed
test jobs. Retry **by job id** (`glab api -X POST projects/:id/jobs/<id>/retry`), sometimes twice. `dependency_scan_govulncheck`
on chat-core is a real blocker that predates the MRs (GO-2026-6505 otlptrace 1.34 → fix MR !133 bumps to 1.45).
Related: [[project_kb100_eval_state_and_rag_direction]], [[reference_chat_core_local_inbound_forward_for_live_tests]].

**2026-10-11 verified state:** chat-core `d0def1a0` (main after AI-295 !131 + AI-296 !132; `deploy_staging_th_arm` on main pipeline
63464 ran 2026-10-10), rag-core-python `1f6c300e` (main after !65 + AI-291 evidence-ingress), agent `6502aee6` (main; no gemini
catalog, no context_budget), FE bundle `index-B3UO1Por.js` with the AI-295/296 UI. Migrations applied: rag-core alembic
`20261006_0014` (read with `kubectl exec <rag-core-python pod> -- sh -lc 'cd /app && .venv/bin/alembic current'`), chat-core
0107/0108/0110 via Job `chat-core-migration-d0def1a0` (the Job manifest in `deployment/job-migration-staging.yaml` with the image
tag swapped). **Read the chat_core `migrations` table with a throwaway pod**: `kubectl run tmp-readonly-… --restart=Never
--image=postgres:16-alpine --overrides=<json with env URI from secretKeyRef sellsuki-chat-core-common-secret/DATABASE_CHAT_CORE_POSTGRESQL_URI>`
then `kubectl logs` and delete it; quote SQL in a JSON file, not inline (the `'` escaping breaks). A 401 from
`port-forward svc/sellsuki-chat-core-svc` on any `/v1/workspaces/*` path proves nothing about routes (auth runs first); use the
FE bundle grep or a logged-in call instead. `tsh` sessions expire after a few days — `tsh login` before kubectl.
