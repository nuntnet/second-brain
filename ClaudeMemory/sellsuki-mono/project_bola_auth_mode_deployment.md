---
name: project_bola_auth_mode_deployment
description: BOLA auth — dual-path AUTH_MODE=kratos (Oathkeeper header-trust + cookie fallback); deployment config in monorepo
metadata: 
  node_type: memory
  type: project
  originSessionId: 8ea4e16e-492a-4d1e-8bcb-146ecab6fad2
---

## Final architecture (2026-06-23)

`AUTH_MODE=kratos` is the single SaaS auth mode — it is dual-path, no separate `gateway` mode:

1. **Header path (prod/Oathkeeper)**: Oathkeeper gateway validates the Kratos session at the edge and injects `X-User-Id` (Kratos identity UUID) + `X-User-Kind`. Backend reads these from middleware, resolves BOLA admin by `KratosIdentityID` (global lookup via `FindAdminsByKratosIdentityIDGlobal`, then workspace-scoped via `GetAdminByKratosIdentityID`). Domain-agnostic — bearyweb.com is fine.
2. **Cookie path (local dev)**: If no `X-User-Id` header (or "anonymous"), falls back to reading the Kratos session cookie and doing whoami. Works on `*.sellsuki.local` (same domain as Kratos cookie).

Discriminator in `isKratosUUID(credential)`: UUID has no `=` or `;`; Kratos Cookie header always does.

**Why:** Ory Oathkeeper is the platform-standard auth pattern used by 8+ backends (management, catalog, customer-book, file-service, i18n, CCS, address). No cookie read, no whoami, no JWT validate in prod. Frontends use `withCredentials` + 401 → redirect to AMS SSO (`VITE_AMS_URL/login?return_to=`).

## Key files changed (commit e806205 on feat/bola-gateway-auth)

- `src/repository/auth_provider/kratos.go` — dual-path CredentialFromRequest + Authenticate
- `src/repository/admin_repository/postgres_gorm.go` — added `GetAdminByKratosIdentityID`, `FindAdminsByKratosIdentityIDGlobal`
- `src/use_case/repository/bola_repository.go` — added both methods to AdminRepository interface
- `src/use_case/repository/mock_admin.go` — mock implementations
- `src/interface/fiber_server/middleware/workspace_auth.go` — pass `x-user-id` and `x-user-kind` headers to auth provider

## Deployment config (lives in this monorepo)

**Backend** `backend/bola-backend/deployment/values-{staging,production}.yml`:
- Add `AUTH_MODE=kratos` (hardcoded value, not a secret)
- Add `ORY_KRATOS_PUBLIC_URL` (for cookie fallback path if needed in staging) — or leave as secret

**Frontend** `frontend/bola-frontend/.gitlab-ci.yml`:
- `.variables_export_staging_frontend` and `.variables_export_production_frontend`
- Add `VITE_AUTH_MODE: 'kratos'`
- Add `VITE_AMS_URL` pointing to Sellsuki AMS (accounts.sellsuki.com) for 401 redirect

**SaaS prod URL**: `bola.bearyweb.com` (prod), `bola-web.staging-th.bearyweb.com` (staging) — **no domain move needed** because Oathkeeper header-trust is domain-agnostic.

## Backfill needed

Existing BOLA admins created via `local_jwt` have empty `kratos_identity_id` column. Must run a backfill script/cmd that: for each admin email → call Kratos admin API `/admin/identities?credentials_identifier=<email>` → get UUID → update `admins.kratos_identity_id`. New admins are set at invite time via `CreateOrFindIdentity`.

## Local dev checklist (all required)

1. backend `.env`: `AUTH_MODE=kratos`
2. frontend must run `vite --mode dev` (loads `.env.dev` with `VITE_AUTH_MODE=kratos`) — Procfile `web-bola` line fixed 2026-06-22 to include `--mode dev`
3. `kratos-ui` service must be running (`accounts.sellsuki.local` → :4455)
4. Clear localStorage when switching modes (`bola_workspace` stale from local_jwt suppresses redirect)

See [[project_overmind_restart_quirk]] for restarting bola-api after env change.
