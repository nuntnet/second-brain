---
name: reference_bola_prod_deploy_mechanics_and_gaps_2026_10_22
description: "BOLA production = git tag vX.Y.Z + manual approve jobs (backend server + every cron, frontend 1), helm --atomic, migrations at boot; prod had been v1.0.29/v1.0.23 (Aug/Jul) while staging ran main; QMS prod v0.1.1 lacks the reservation RPCs BOLA-187 calls; chart secretKeyRef is never optional"
metadata:
  node_type: memory
  type: reference
  originSessionId: e2d40730-b4e0-484b-b3c2-d83461734bf6
  modified: 2026-10-22T07:47:13.077Z
---

Learned 2026-10-22 while preparing the first BOLA production release since August (full plan: `docs/release/bola-prod-release-plan-2026-10-22.md` in the monorepo).

- **Trigger:** only a tag `v\d{1,2}\.\d{1,2}\.\d{1,2}` on main runs the production pipeline (`.rules_check_production`); the deploy stage jobs are `when: manual` (`.rules_approve_production`). Backend needs **every** job clicked — `deploy_production_amd` + one `deploy_cron_production_amd` per cron (4 before BOLA-333, 5 after: broadcast, lon-job, rich-menu-sync, segment-refresh, line-oa-health). Frontend: `deploy_production` only. Tags are lightweight tags on the merge commit, created by nuntawit.
- **helm upgrade runs with `--atomic --wait`** → a pod that never becomes ready rolls back by itself. `helm-generic-chart/templates/deployment.yaml` renders `secretKeyRef` WITHOUT `optional: true`, so a values file that references a Vault key the environment lacks = `CreateContainerConfigError` = rollback. Check `git diff <prod-tag> origin/main -- deployment/values-production.yml | grep key:` before every tag.
- **bola-backend migrations run at boot** (`runMigrationsOnStartup` → Fatal), not in an init container (that is only the self-host tenant chart). Migrations already applied before the failing one do not roll back.
- **QMS production is `v0.1.1` (2025-07-15)** and has no `InternalReserveQuota / CommitReservation / ReleaseReservation / AdjustQuota` — the RPCs bola-backend's BOLA-187 code calls, fail-closed, whenever `ws.CompanyID != ""`. Safe today only because nothing on prod can create a company-bound workspace (CCS prod v1.6.1 and backoffice-api prod v1.11.0 have no BOLA_API_BASE_URL). **QMS must go to prod before CCS/OC2Plus start binding workspaces there.** `QUOTA_ENFORCEMENT_DISABLED=true` is refused at boot in production (OC-4267).
- **Drift found:** prod had been on backend `v1.0.29` (2026-08-05) / frontend `v1.0.23` (2026-07-21) while staging ran `main` HEAD — 205 / 70 commits apart. Staging's 4 cron deploy jobs on the latest main pipeline were never clicked, so staging crons ran an older image than the server.
- **User decision 2026-10-22:** promote every develop-only card to main first (stacked MR wave, see [[reference_stacked_promotion_wave_develop_to_main]]), then tag backend `v1.0.30` / frontend `v1.0.24`.

Related: [[project_bola_deploy_topology]], [[project_bola_deploy_values_in_repo]], [[reference_manual_staging_gate_silent_drift]].

**Outcome 2026-10-23:** released — backend `v1.0.30` (3c58e4e4, helm rev 44) + frontend `v1.0.24` (6864d2e7). Every production-touching step (git tag push, playing deploy jobs, `kubectl` reads on the prod context, Vault writes) was refused by the auto-mode classifier ("Production Deploy" / "Production Reads") — the user ran each one from their own shell while the agent watched GitLab's API (pipelines, jobs, deployments are readable). Vault prod host `vault.internal.production.sellsuki.com` needs the WireGuard profile `sellsuki-production`; `vault kv patch -mount=kubernetes-secret <ns>/<secret> KEY=…` then `kubectl annotate externalsecret … force-sync=$(date +%s)`. Kafka prod has `auto.create.topics.enable=true`, so new topics need no SRE ticket.
