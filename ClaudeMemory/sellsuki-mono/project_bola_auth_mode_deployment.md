---
name: project_bola_auth_mode_deployment
description: "BOLA auth — SaaS deployment uses kratos, multi-tenant uses local_jwt; env lives in SRE repo not monorepo"
metadata: 
  node_type: memory
  type: project
  originSessionId: 42bf1a19-7f07-405b-a1a4-451da110e784
---

BOLA auth provider is chosen at runtime by env var, no code change needed (code fully supports all modes):
- backend `AUTH_MODE` (`backend/bola-backend/.env`, parsed in `cmd/bola_server/main.go`, provider built in `cmd/bola_server/helper.go:601`): `kratos` | `local_jwt` (default) | `oidc` | `header`
- frontend `VITE_AUTH_MODE` (`frontend/bola-frontend/.env.dev`): `kratos` | `local_jwt`

**Intended mapping:**
- **SaaS** deployment (`bola.sellsuki.com`) → **kratos** (central Sellsuki Ory Kratos SSO, session cookie)
- **multi-tenant / self-hosted** deployment → **local_jwt** (BOLA issues its own JWT)

kratos provider resolves the BOLA admin by the Kratos identity's **email** — so a bola DB admin record must exist with email matching the Kratos identity, else `admin_not_found`.

**SaaS prod URL is `bola.bearyweb.com`** (staging `bola-web.staging-th.bearyweb.com`), NOT bola.sellsuki.com. **Decision (2026-06-22): move SaaS to `*.sellsuki.com`** (Option A) so the central Kratos cookie SSO works — because a Kratos session cookie scoped to `.sellsuki.com` is NEVER sent to a `bearyweb.com` page (different registrable domain) and bearyweb isn't in Kratos's `WHITELIST_DOMAIN`/`allowed_return_urls` (which cover `*.sellsuki.com`/`*.patona.online`/`*.oc2.plus`). This cross-domain cookie issue is the real reason SaaS auth was "wrong" — not just a missing env.

In-cluster Kratos URLs (from `backend/kratos-ui-go/deployment/values-*.yml`): public `http://kratos-public`, admin `http://kratos-admin:80` (no token), browser UI `https://accounts[.staging-th].sellsuki.com`. BOLA kratos provider forwards the whole Cookie header to whoami, so it doesn't need the cookie NAME (sellsuki_session / sellsuki_session_staging).

**Deploy config DOES live in this monorepo** (corrects earlier note): backend `backend/bola-backend/deployment/values-{staging,production}.yml` (helm env list) and frontend `frontend/bola-frontend/.gitlab-ci.yml` (`.variables_export_{production,staging}_frontend` build-time VITE_* vars). Work started on branch **`feat/bola-saas-kratos-sso`** in BOTH submodules (committed, NOT pushed/merged) — adds AUTH_MODE=kratos + Kratos URLs + moves domains to *.sellsuki.com, with TODO(SRE) on CloudFront/ACM IDs. Must cut over atomically with SRE (flipping kratos on the live bearyweb domains alone = 401). The backend CI (`backend/bola-backend/.gitlab-ci.yml`) only includes SRE pipeline templates and has no env.

**(superseded) Deployment env vars are NOT in this monorepo** — `.gitlab-ci.yml` includes pipelines from `sellsuki/sre/deployment/pipeline-deployment` (GitLab). To fix prod auth, the SaaS env must be set there: backend `AUTH_MODE=kratos` + `ORY_KRATOS_PUBLIC_URL` (+ `KRATOS_ADMIN_URL`/`KRATOS_ADMIN_TOKEN` for invite/recovery); frontend `VITE_AUTH_MODE=kratos` + `VITE_KRATOS_LOGIN_URL`/`VITE_KRATOS_ACCOUNTS_URL`.

Local dev: Kratos runs in docker-compose (public :4433), `accounts.sellsuki.local` = kratos-ui-go login UI (overmind `kratos-ui`, :4455). See [[project_overmind_restart_quirk]] for restarting bola-api after env change.

**To test BOLA kratos mode locally, ALL of these must hold (each was a separate gotcha):**
1. backend `.env` `AUTH_MODE=kratos` (default is local_jwt)
2. frontend must run as `vite --mode dev` so `.env.dev` (`VITE_AUTH_MODE=kratos`) loads — plain `bun run dev`/`vite` uses mode `development` which loads `.env`/`.env.development` (neither sets the var) → silently falls back to local_jwt. The main `Procfile` web-bola line was missing `--mode dev` (fixed 2026-06-22; `Procfile.bola`/`Procfile.frontend` already had it).
3. `kratos-ui` service must be running (`accounts.sellsuki.local` → :4455) or login redirect gives 502. Starting only `bola-api,web-bola` in overmind is not enough — include `kratos-ui`.
4. clear localStorage when switching modes: kratos `isAuthenticated()` only checks `getWorkspaceId()`, so a stale `bola_workspace` from a local_jwt session suppresses the redirect.
