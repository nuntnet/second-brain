---
name: reference-env-urls
description: Sellsuki public URL convention per environment (dev/staging/prod) + API gateway path routing
metadata: 
  node_type: memory
  type: reference
  originSessionId: 06c2449b-af9f-42b9-bf19-e32fb331ac0f
---

Sellsuki environment URL convention (confirmed from frontend repo configs, 2026-06-25):

**Env suffix:** dev = `.dev-th` · staging = `.staging-th` · prod = `(none)`.

**API gateway (Oathkeeper)** routes per-service by path:
`api.{suffix}.sellsuki.com/{service}/v1` — e.g. `api.dev-th.sellsuki.com/central-configuration-system/v1`, `/i18n-management/v1`, `/pis`; prod `api.sellsuki.com/ccs/v1`, `/file/access/private`.

**Confirmed anchors:**
- Auth / AMS (Kratos login UI, served by kratos-ui-go): `accounts.sellsuki.com` (prod) · `accounts.staging-th.sellsuki.com` · `accounts.dev-th.sellsuki.com` · local `accounts.sellsuki.local:4455`. (matches [[sellsuki-auth-pattern]])
- Admin console: `admin.dev-th.sellsuki.com` (also `admin.dev.sellsuki.com` seen); prod `admin.sellsuki.com` (convention, unconfirmed).
- PIS: `pis.dev-th.sellsuki.com` / `pis.staging-th.sellsuki.com` (also legacy `pis.dev.sellsuki.com`).
- Sellsuki RAG: dev gateway `api-rag.dev-th.sellsuki.com` · staging UI `rag.staging.sellsuki.com`. (see [[project-control-tower]])
- GitLab: `gitlab.sellsuki.com`. Internal MCP: `mcp-{atlassian,gitlab,outline}.internal.production.sellsuki.com`.

**Gotchas:** suffix is inconsistent across services/eras — some legacy services use bare `.dev`/`.staging` (file.dev.sellsuki.com) or dedicated per-service hosts (`api.oms.sellsuki.com`, `api.customerbook.sellsuki.com`) instead of the unified gateway path. Local dev = `*.sellsuki.local` via Caddy (full port map in `.claude/knowledge/domain-routing.md`).
