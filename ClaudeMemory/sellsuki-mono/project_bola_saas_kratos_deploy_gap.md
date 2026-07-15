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

**rgb72 (self-host) unaffected:** local_jwt, no AUTH_MODE=kratos, no ORY_KRATOS_PUBLIC_URL ref → safe to bump to v1.0.24. See [[project_bola_auth_mode_deployment]], [[project_bola_kratos_sso_staging]].
