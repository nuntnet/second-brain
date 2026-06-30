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

**Backend whoami gotcha (token_invalid loop):** `/v1/me/workspaces` (handleGetMyWorkspaces) deliberately uses the COOKIE path (resolve by email) — it omits `x-user-id` from its headers map, unlike AdminGuard/FlatAdminGuard which pass `x-user-id` (header-trust). So this endpoint makes the backend call `{ORY_KRATOS_PUBLIC_URL}/sessions/whoami` itself. CRITICAL: `accounts.staging-th.sellsuki.com` routes to **kratos-ui-go-svc.share** (the UI app), NOT the Kratos public API — it 404s "Cannot GET /sessions/whoami" (Kratos serve.public.base_url is set to that host but Emissary routes it to the UI). So `ORY_KRATOS_PUBLIC_URL` must be the internal service **`http://kratos-public.share`** (same one Oathkeeper uses, oathkeeper-values.yaml check_session_url), NOT the public accounts URL. If it's set to the accounts URL → whoami 404 → ErrAuthTokenInvalid → `{"error_code":"token_invalid"}` 401 → frontend kratos-mode redirect loop. Secret: `bola-backoffice-secret` key `ORY_KRATOS_PUBLIC_URL` (k8s secret, not in git).

**Identity resolution gotcha (POST create-workspace 504):** `ORY_KRATOS_ADMIN_URL` is UNSET in `bola-backoffice-secret` on staging-th → the kratos provider falls back to the public URL for admin-API calls (`/admin/identities/{id}`), but the public service (`kratos-public.share`) has no `/admin` route, and the provider's `http.Client` had no timeout → calls hang ~30s → Emissary (timeout_ms 30000) 504s. Resolve a kratos identity to email via the COOKIE → `/sessions/whoami` (public URL, always works, returns id+email) rather than the admin API. Fixed in MR !102 (cookie-first resolution + 15s kratos HTTP timeout). NOTE: invite flows (`CreateOrFindIdentity`, `GenerateRecoveryLink`) DO need the admin API — set `ORY_KRATOS_ADMIN_URL=http://kratos-admin.share:4434` in the secret for those to work.

**Split-host frontend gotchas (kratos staging):** the SPA is on `bola-web.staging-th.bearyweb.com` but the API on `bola-api.staging-th.sellsuki.com` — DIFFERENT hosts. Consequences hit repeatedly: (1) any **relative `fetch()`** (not via the `api` singleton / without `VITE_API_URL`) hits the static SPA host → returns `index.html` (200 in dev via proxy, 404 on staging) → `res.json()` "Unexpected token '<'". Always prepend `VITE_API_URL`. (2) The `api` client's **global 401 handler clears `bola_workspace` + redirects to Kratos** — so ANY 401 (even a cosmetic header fetch like WorkspaceMenu's `GET /v1/workspaces/:id`) causes a logout/redirect LOOP (dashboard→blank→dashboard). (3) Backend readiness is `/system/readiness` (not `/readiness`). (4) Vite dev proxy needs a `/system` rule.

**Backend auth gotchas:** (a) Workspace routes (`/v1/workspaces/:id`, getWorkspaceBySlug, updateWorkspace, outbound-webhook) authenticate via an INLINE `requireFlatAuth` in `route/v1/workspace/route.go`, NOT the AdminGuard/FlatAdminGuard middleware — easy to miss; it must read `x-user-id` + `cookie` for kratos. (b) Durable identity-resolution pattern (no admin-API dependency): AdminGuard/FlatAdminGuard/requireFlatAuth, on header-trust `ErrAdminNotFound`, retry `Authenticate(cookie)` → `/sessions/whoami`; `authenticateByCookie` links `kratos_identity_id` on first cookie login. This self-heals email-seeded admins without needing `ORY_KRATOS_ADMIN_URL`.

**Cross-tenant leak (fixed):** `customer_repository.ListCustomers/CountCustomers` historically scoped by `line_oa_id` ONLY — empty line_oa = unscoped dump of ALL followers across workspaces (a new empty workspace showed 471k contacts). Now takes `workspaceID` with a guard: both empty → return nothing. Handler sources workspace from the authenticated admin, never a client query param. Keep this invariant on any new customer/follower query.

Related: [[project_bola_auth_mode_deployment]] [[project_bola_migrations_jsonb]]
