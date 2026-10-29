---
name: reference_local_usage_routes_cannot_show_allow_cells
description: "Local stack cannot exercise the ALLOW side of chat-core /v1/companies/:id/usage|usage/cost: local rps has no chat_workspace.usage.read/billing.read (0024 not applied), ai-agent local has no QMS (/usage 502) and no usage analytics — and on ai-agent main before AI-232's fix, /internal/v1/usage/cost with no analytics PANICS the agent. Service token header is X-Service-Token."
metadata:
  node_type: memory
  type: reference
  originSessionId: 55f225da-3e5e-4b6f-9508-e655e398936e
  modified: 2026-10-29T15:29:12.810Z
---

Seen 2026-10-29 (AI-232/AI-230, two verifiers).

- Local rps `permission_lists` has no `chat_workspace.usage.read` / `billing.read`; no role carries them. The seed identity
  (company_admin of 85fa00dd) gets 403 on `/usage` and `/usage/cost`. Only the deny half of a usage matrix is observable
  locally; the allow half (Owner sees zeroed cost, provider sees cost, audit line) is tier B on dev.
- ai-agent local: `/internal/v1/usage` → 502 (no QMS); `/internal/v1/usage/cost` → nil-pointer panic that kills the
  process (typed-nil `*usage_analytics.Service` passes the `configured()` guard). Fixed on
  `fix/AI-232-usage-analytics-unconfigured-503` (`5c9644f`) → 503 `usage_analytics_not_configured`. Until that merges,
  do not curl that route on a stack without it.
- ai-agent internal routes take `X-Service-Token: <SERVICE_TOKEN from Procfile.ai-mvp ai-agent line>`; `Authorization: Bearer` → 401.
- To show console rendering without data, the verifier stubbed `window.fetch` in the tab with payloads shaped like
  ai-agent `CompanyReport` — rendering evidence only.

Related: [[reference_local_ai_chat_console_open_on_localhost_with_seed_identity]], [[reference_overmind_restart_dead_process_ai_mvp_socket]].
