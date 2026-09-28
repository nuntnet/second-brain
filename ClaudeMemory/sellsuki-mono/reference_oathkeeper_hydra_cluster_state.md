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

**How to add an MCP ingress** (Pan, 2026-09-28, with his examples):
- **Rule:** open an MR in `sellsuki/sre/configuration/<product>`, file `manifest/<env>/rule.yaml`. OC2Plus rules live in `sellsuki/sre/configuration/oc2plus`, which is ArgoCD `octoplus-development-manifest`. Pattern: `sre/configuration/sellsuki` !23 — a `-wellknown` rule (anonymous) plus a `/mcp` rule (oauth2_introspection).
- **Host:** open an MR in `sellsuki/sre/configuration/api-gateway` adding an Emissary `Host` + `Mapping` to `oathkeeper-proxy.share:4455` (pattern: !222). SRE then maps the DNS.
- **Watch line endings:** `staging-th/rule.yaml` in the oc2plus repo is CRLF. Writing it through Python's text mode rewrites the whole file.
- **Wildcard certs already exist:** `*.crm.dev-th.oc2.plus` and `*.crm.staging-th.oc2.plus`.
- **Hydra clients** are registered by hand through the admin ingress `hydra-admin-ingress.share[-dev].internal.staging-th.sellsuki.com` (VPN, no auth). There is no maester. Doc: docs.sellsuki.com/doc/setup-hydra-6PZrU6jvFt.
- **Hydra port wildcarding:** only IP-literal loopback (`127.0.0.1`, `[::1]`) gets any port; `localhost` gets none. Without `audience`, the first refresh returns 400.
- **Codex/ChatGPT on dev:** kratos-ui-go on development-th is behind and lacks RFC 9207, so these clients cannot log in on dev.
