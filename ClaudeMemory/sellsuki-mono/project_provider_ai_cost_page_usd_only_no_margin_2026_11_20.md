---
name: project_provider_ai_cost_page_usd_only_no_margin_2026_11_20
description: "CCS2 /ai/usage (AI-234/AI-112) shows LLM cost per company in USD only — no revenue, no THB, no margin; QMS holds units not money by contract"
metadata:
  node_type: memory
  type: project
  originSessionId: 58c6eca6-d6eb-446b-bf6a-a91ba8b63329
  modified: 2026-11-20T06:21:48.159Z
---

Provider-admin AI cost page exists: `frontend/sellsuki-provider-management-frontend` route `/ai/usage` (Usage.svelte, AI-234 on main), via management-backend `GetProviderAIUsage` → ai-agent `GET /usage` (ledger `ai_usage_events` in data-pipeline). Charge-class breakdown (AI-112) is on develop only as of 2026-11-20.

No plan price/revenue join, no FX/THB, no margin column anywhere. `docs/ai-e5-qms-contract-ask.md` fixes the boundary: QMS = units, AI side = cost/margin. Margin card is AI-461 (To Do); AI-202 (AI package ↔ commercial plan mapping) is its prerequisite.

**Why:** "do we profit on AI" needs revenue that no service stores yet.
**How to apply:** a margin view = new price source + FX + join, not a UI tweak on the existing page.
