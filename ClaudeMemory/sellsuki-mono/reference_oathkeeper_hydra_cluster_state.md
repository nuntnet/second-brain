---
name: reference-oathkeeper-hydra-cluster-state
description: Oathkeeper rules are CRDs on the cluster, not in the repo. Backoffice accepts cookies only. Hydra introspection is already used (rag-core-mcp rule). No NetworkPolicy in octoplus. Checked 2026-09-27.
metadata:
  type: reference
---
Checked 2026-09-27 on the staging-th cluster, read-only via kubectl. Production was not checked because the tsh login had expired.

- **Oathkeeper**: a single instance in ns `share` serves both dev and staging. Rules are CRDs (`rules.oathkeeper.ory.sh`) living in each service's namespace. SRE manages them and they are not in any repo. Read them with `kubectl get rules.oathkeeper.ory.sh -A`. Global config is in `cm/oathkeeper-config` (ns `share`).
- **Backoffice rule** `oc2plus-crm-backoffice-backend-auth` (octoplus-dev / octoplus):
  - Matches `api.crm.<env>.oc2.plus/backoffice<.*>`.
  - Authenticators: `cookie_session` (Kratos whoami), then `anonymous`.
  - Header mutator sets `X-User-Id={{.Subject}}`.
  - A bearer token reaches the API as anonymous and gets a 401, so it fails closed.
- **Hydra** runs in `share-dev` (dev) and `share` (staging). The `oauth2_introspection` authenticator is already in use by expccs, posh openapi, sellsuki openapi and **`rag-core-mcp`**. That last rule was created 2026-09-15 for host `rag-mcp.<env>.sellsuki.com/mcp`. Its mutator also sets X-Client-Id, X-Scope and X-User-Email, so it is the template for any new MCP. No MCP code was found in the rag-core repo on any branch. There are no OAuth2Client CRDs; clients must have been created some other way.
- **No NetworkPolicy** in `octoplus` or `octoplus-dev`. Any pod in the cluster can call backoffice-api-svc directly with a forged `X-User-Id`.
- Oathkeeper requires exactly one rule per URL. A new path under `/backoffice` will collide with the existing `/backoffice<.*>` regex, so use a separate host.

Related: [[project-oc2plus-mcp-assistant-idea]] · [[reference_dev_th_cluster_access]]
