---
name: project-oc2plus-mcp-assistant-idea
description: "OC2Plus backoffice MCP (OC-4625, epic OC-4624) — built and live on dev since 2026-09-28; 23 read-only tools in backoffice-api /mcp, host mcp.crm.<env>.oc2.plus; what is done, what is open"
metadata:
  node_type: memory
  type: project
  originSessionId: 4341156f-9031-4920-83b6-fd7bcbf45cd4
  modified: 2026-09-28T23:06:05.181Z
---

**Built, no longer an idea.** OC-4625 = MCP v1 read-only, 23 tools, route `/mcp` inside `backend/oc2plus-line-crm-service-backoffice-api` (package `mcproute`, `src/interface/fiber_server/mcp/`). Merged !646 → develop `193067ef` 2026-09-28, deployed to development-th that evening (`MCP_ENABLED` "true" only in values-development; staging/prod "false").

Infra that exists: Oathkeeper rules (sre/configuration/oc2plus !9), gateway host (sre/configuration/api-gateway !225), DNS `mcp.crm.dev-th.oc2.plus` / `mcp.crm.staging-th.oc2.plus`, Hydra public client `sellsuki-oc2plus-mcp` on **dev and staging** (files `deployment/hydra/{development,staging}-th.json`, incl. `http://localhost:3119/callback` for Claude Code; the user runs Hydra writes — auto mode blocks Claude). Plugin for Claude Code + Codex: connectors MR !3 (callback port 3119), awaiting RAG team review.

**Why:** OC2Plus is hard to use; staff ask the AI they already use, with their own permissions. Design reasoning (in backoffice-api, not ai-agent/kit; forward the user's identity via Oathkeeper headers; per-tool Keto check reused by calling the REST routes in-process) is in the ledger `docs/cards/OC-4625.md` decisions table — do not re-derive.

**How to apply:**
- Live-dev capture 2026-09-28 found 87 confirmed defects (member_search phone/email, check_my_permission missing 5 codes, member_code contains-match, filter_point_id, request logger writing MCP bodies with PII…). Fixed on branch `fix/oc-4625-mcp-live-findings` (2026-09-29).
- AC-C6 added 2026-09-28: backoffice FE page "เชื่อมต่อ AI" `/connect-ai` (env `VITE_MCP_URL`, dev only). User chose NOT to build "list/revoke connected AI apps" (needs Hydra admin API) — out of scope.
- **State 2026-09-29 20:10:** every V1 AC passes on dev except **B4 (deferred by the user — no non-owner test account)**. A5 passed after kratos-ui-go !195 (RFC 9207 to share-dev, merged by user OK) — Codex CLI 0.159.0 signs in and calls tools. C4: connectors !3 merged, but the GitHub marketplace mirror is still 404 (RAG team). C6 page live at crm.dev.oc2.plus/connect-ai (FE !679); !681 turns off its Codex warning. Pending: human sign-off of the ACs + Q18 (B9 metric). Side fix outside V1: CCS !378 (Owner preset news/coupon).
- Production is blocked on the PII/DPA question (member data goes to Anthropic/OpenAI). Do not flip MCP_ENABLED in production.
- Related: [[reference_oathkeeper_hydra_cluster_state]] [[reference_rag_core_mcp_prior_art]] [[feedback_no_tiny_cards_bundle_as_ac]]
