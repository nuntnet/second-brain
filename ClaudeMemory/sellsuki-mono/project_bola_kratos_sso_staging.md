---
name: project_bola_kratos_sso_staging
description: BOLA staging Kratos SSO — API must be on *.staging-th.sellsuki.com (cookie domain); Emissary config repo + deploy gating
metadata: 
  node_type: memory
  type: project
  originSessionId: 8ea4e16e-492a-4d1e-8bcb-146ecab6fad2
---

Enabling Kratos SSO for BOLA staging (`bola-web.staging-th.bearyweb.com`). Login redirect loop root cause + fix:

**Root cause:** Kratos session cookie (`ory-helm/staging-th/kratos-values.yaml`) is `domain: staging-th.sellsuki.com`, `same_site: None`, name `sellsuki_session_staging`. Browser will NEVER send it to a `*.bearyweb.com` host (different root domain — unrelated to SameSite). So the BOLA API must live on `*.staging-th.sellsuki.com`. Because `same_site: None`, the **frontend can stay on bearyweb.com** (cross-site fetch with credentials works).

**Where Emissary Host/Mapping live:** `gitlab.sellsuki.com/sellsuki/sre/configuration/api-gateway` repo (NOT the `emissary` repo, NOT `terraform`). Per-env dirs `staging-th/`, `production/`, etc. Files: `hostnames-bola.yaml` (kind: Host, TLS) + `mapping.bola.yaml` (kind: Mapping, routing). SRE-owned → open MR, don't push main. Standard auth route: `service: oathkeeper-proxy.share:4455` (Oathkeeper validates cookie, injects `X-User-Id`). Wildcard cert secret `staging-th.sellsuki.com` already exists.

**The fix (all done):**
- api-gateway!209 (merged): added Host+Mapping `bola-api.staging-th.sellsuki.com` → oathkeeper-proxy.share:4455
- bridge.git: Oathkeeper Rule `bola-api-auth` matching that hostname (pushed)
- ory-helm: oathkeeper CORS allows `*.staging-th.bearyweb.com` (pushed)
- bola-backend staging: `AUTH_MODE=kratos` + `ORY_KRATOS_PUBLIC_URL` already set
- frontend MR !75: staging `VITE_API_URL` → `https://bola-api.staging-th.sellsuki.com` (was broken `internal.staging-th.sellsuki.com` = untrusted internal-CA cert → ERR_CERT_AUTHORITY_INVALID)

**Frontend deploy gating (non-obvious):** `back-office-of-line-api-frontend` staging build+deploy runs ONLY on `main` (or tag `vX.Y.Z-staging`) per `pipeline-deployment/templates/rules.template.yml` `.rules_check_staging`. `deploy_staging` job is `when: manual`. Feature-branch / MR-ref pushes do NOT deploy staging. So a CI-var change must land on `main` then someone clicks the manual deploy_staging job. Staging uses `main`'s `.gitlab-ci.yml`.

Related: [[project_bola_auth_mode_deployment]]
