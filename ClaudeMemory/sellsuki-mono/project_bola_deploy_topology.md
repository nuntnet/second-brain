---
name: project_bola_deploy_topology
description: BOLA deploy topology — rgb72 self-host tenant via SRE bola-tenants chart (migrate-ci init container); FE tenants repo; kratos-admin port-80 gotcha
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-07-22T17:15:30.991Z
---

BOLA has **two distinct deploy paths** with different Helm charts (learned by grounding against live prod/staging + the SRE repos, 2026-07):

**1. Repo-chart path (staging + SaaS `bola` ns)** — deploys from bola-backend's own `deployment/values-{staging,production}.yml` via the SRE golang-th pipeline. Migrations run **on boot** (Fatal→crashloop) — see [[project_bola_migrations_jsonb]]. Staging = separate `eks_staging_th-cluster`, ns `bola`, **auto-deploys on `main`** (`deploy_staging_th_arm`). AUTH_MODE=kratos.

**2. Self-host tenant path (rgb72 etc.)** — deploys via SRE repo **`sellsuki/sre/configuration/bola-tenants`**, file `chart/tenants/{tenant}.yaml` (e.g. `rgb72.yaml`: `bolaServer.tag` **pinned**, `bolaAuthProxy.allowOrigins`, `webhookBaseURL`, `autoPushWorker.enabled`). This chart runs migrations in a **`migrate-ci` INIT container** (NOT server boot) → a bad migration = new pod **stuck in Init, OLD pod keeps serving**, rolling update stalls, **no crashloop/outage** (safer than the repo-chart boot path). Check migration success via `kubectl -n bola-{tenant} logs deploy/bola-server -c migrate-ci`.
- **rgb72** = Thai LINE OA self-host tenant, `AUTH_MODE=local_jwt`, ns `bola-rgb72` (prod cluster), domain `lon.xn--12car7cvaebaj4dgahdhd8ac4bzmub2b2a3hxhgoz8rrag.com`. autoPushWorker held disabled (SRE-1681).

**Frontend tenants** — separate repo **`sellsuki/bola/bola-frontend-tenants`**, file `tenants/production/{tenant}.env`. No `FRONTEND_REF` line = build.sh **auto-resolves the latest `vX.X.X` tag** at pipeline time (add `FRONTEND_REF=` to sticky-pin). Deploy via **manual "Run pipeline"** with var `TENANT={tenant}` (or on `.env` change); the `deploy:{tenant}` job is a **manual gate** (a plain "Run pipeline" only builds — must also click deploy). CloudFront distro is **customer-managed** (e.g. rgb72 `E2MELTMNL3J5CW`, acct 172823157975) — SRE only s3-syncs + invalidates, never alters the distro/bucket/cert.

**⚠️ kratos-admin port-80 gotcha (staging invite/provision):** `KRATOS_ADMIN_URL` must be `http://kratos-admin.share` (**port 80** — ClusterIP routes only 80, forwards to container 4434). Hardcoding `:4434` → dial hangs → 15s `Client.Timeout` on `POST /admin/identities` → invite fails. Fixed on main (`values-staging.yml`) but staging must redeploy to pick it up. Related: [[project_bola_auth_mode_deployment]], [[project_bola_kratos_sso_staging]], [[project_bola_rbac_keto_direct]].
