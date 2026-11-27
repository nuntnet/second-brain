---
name: project-ai-chat-dashboard-and-usage-need-data-pipeline-prod
description: "Console Dashboard 503 + Usage cost page both need ai_chat_data_pipeline dashboard-api, which is not on prod and is deny-all everywhere (2026-11-20, AI-498)"
metadata:
  node_type: memory
  type: project
  originSessionId: d0da5cb3-1de8-4ef2-9e7c-5c5085687b5d
  modified: 2026-11-20T14:50:54.775Z
---

As of 2026-11-20 (AI-498): the AI chat console Dashboard (`dashboard_not_configured` 503 from chat-core, `DASHBOARD_API_BASE_URL` unset) and the ai-agent Usage cost page (`AI_USAGE_ANALYTICS_URL`) both read the same `cmd/dashboard-api` of gitlab `sellsuki/data-pipeline/ai_chat_data_pipeline` (another team's repo). It runs only on staging (`ai-chat-usage-dashboard-svc.data-pipeline-dev:8100`); its CI has no production deploy job, prod has no such service.

`/v1/dashboard` uses `DenyAllAccessChecker` unless `DASHBOARD_ENABLE_CCS_ACCESS_CHECK=true`, and the CCS endpoint it would call (`/internal/v1/workspace-access`) does not exist. So setting chat-core's `DASHBOARD_API_BASE_URL` turns the 503 into a 403 for every owner — not a values fix.

**Why:** nunt said 2026-11-20 prod needs both pages.
**How to apply:** a fix needs code (dashboard-api trusted-caller mode on `DASHBOARD_SERVICE_TOKEN` + chat-core client sending it) and a prod deploy of the pipeline. Ledger: docs/cards/AI-498.md. Related [[project-provider-ai-cost-page-usd-only-no-margin]].

**Update 2026-11-27:** nunt approved splitting the remaining work: AI-512 (dashboard-api trusted caller on the `DASHBOARD_SERVICE_TOKEN` bearer + chat-core sends `DASHBOARD_API_SERVICE_TOKEN`) and AI-513 (pipeline production values + CI deploy jobs, namespace `data-pipeline` assumed). Pipeline MRs !15 and !16 were opened in the data team's repo as proposals for them to merge. Ledgers: docs/cards/AI-512.md, AI-513.md.
