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

**Deployment env vars are NOT in this monorepo** — `.gitlab-ci.yml` includes pipelines from `sellsuki/sre/deployment/pipeline-deployment` (GitLab). To fix prod auth, the SaaS env must be set there: backend `AUTH_MODE=kratos` + `ORY_KRATOS_PUBLIC_URL` (+ `KRATOS_ADMIN_URL`/`KRATOS_ADMIN_TOKEN` for invite/recovery); frontend `VITE_AUTH_MODE=kratos` + `VITE_KRATOS_LOGIN_URL`/`VITE_KRATOS_ACCOUNTS_URL`.

Local dev: Kratos runs in docker-compose (public :4433), `accounts.sellsuki.local` = kratos-ui-go login UI. See [[project_overmind_restart_quirk]] for restarting bola-api after env change.
