---
name: project_bola_saas_kratos_deploy_gap
description: BOLA SaaS v1.0.24 deploy failed — AUTH_MODE=kratos needs ORY_KRATOS_PUBLIC_URL key in Vault; worker-vs-server split is the diagnostic lever
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
---

BOLA SaaS (`bola` ns) v1.0.24 helm deploy failed 2026-07-15. Root cause = **`CreateContainerConfigError: couldn't find key ORY_KRATOS_PUBLIC_URL in Secret bola/bola-backoffice-secret`** — container never started (NOT a crash, NOT migration).

v1.0.24 is the **first release flipping SaaS to `AUTH_MODE=kratos`** (commit `7410233`). Its deployment adds env `ORY_KRATOS_PUBLIC_URL` via secretKeyRef, but the key was never added to the Vault source. `initAuthProvider` case "kratos" (cmd/bola_server/helper.go:616) reads `os.Getenv("ORY_KRATOS_PUBLIC_URL")`.

**Fix:** add key `ORY_KRATOS_PUBLIC_URL` to Vault path `kubernetes-secret/bola/bola-backoffice-secret` (ExternalSecret `bola-backoffice-secret` → ClusterSecretStore `vault-backend`, mergePolicy Replace, refresh 300s). Value = same prod Kratos public API base CCS/kratos-ui use (likely `https://accounts.sellsuki.com` — VERIFY, don't guess). Then force-sync ExternalSecret + re-deploy. Only ONE key missing (other 27 present).

**Diagnostic lever (reusable):** bola runs migrations on boot in `setupGorm`→`runMigrationsOnStartup`, called by `initRepositories` for **BOTH server AND worker** (main.go:180, before mode dispatch). So cron/worker pods (`--mode=worker`) exercise DB+migrations+Redis but SKIP `initInterfaces`/auth/server-only env. **Worker Completed + server failing ⇒ problem is server-mode-only (auth/env/route/HTTP), NOT migrations/DB/data.** This proved migrations 0118–0121 safe & already applied. See [[project_bola_migrations_jsonb]].

No downtime: deployment maxUnavailable=0/maxSurge=100% + helm --atomic → old v1.0.23 served throughout, clean rollback.

**rgb72 (self-host) unaffected:** local_jwt, no AUTH_MODE=kratos, no ORY_KRATOS_PUBLIC_URL ref → safe to bump to v1.0.24 (done via bola-tenants MR !8, chart/tenants/rgb72.yaml tag→v1.0.24). See [[project_bola_auth_mode_deployment]], [[project_bola_kratos_sso_staging]].

**Follow-on SSO allowlist fix (prod):** after v1.0.24 (kratos) came up, login on `https://bola.bearyweb.com` → `return_to URL not allowed`. Prod shared Kratos (`share/kratos-config`, ArgoCD `ory-kratos-production`, MANUAL sync, values repo `sellsuki/share/ory-helm` file `production/kratos-values.yaml`) allowlist had only patona.online/*.sellsuki.com/*.oc2.plus. Fix = ory-helm MR !70: add `https://bola.bearyweb.com/` to `selfservice.allowed_return_urls` + `serve.public.cors.allowed_origins` (kratos) AND `https://bola.bearyweb.com` to `production/oathkeeper-values.yaml` proxy `cors.allowed_origins` (ArgoCD `ory-oathkeeper-production`, was already OutOfSync). Same pattern as staging commits d91905e + 1f86365. After merge SRE must sync BOTH argocd apps; kratos config-checksum rolls pods. Architectural gotcha (from [[project_bola_kratos_sso_staging]]): cookie domain=sellsuki.com so BOLA API must be on `*.sellsuki.com` (frontend may stay on bearyweb via same_site:None) — verify SaaS VITE_API_URL.
