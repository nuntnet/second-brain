---
name: chatcore-is-the-admin-bff
description: "chat-core is already the AI admin panel's browser-facing BFF; it owns 7 scattered template stores, no companies table, and rag-core has no service actor"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 01c88554-c2ad-4379-b428-9806eeccfe92
  modified: 2026-08-28T03:12:19.693Z
---

Measured 2026-08-28 by reading chat-core, rag-core and CCS source plus the local
`chat_core` DB — three "nobody owns this yet" premises on the AI board turned out
to be stale.

**chat-core is the browser-facing admin API, and already a BFF.** 12+ route groups
shaped `/v1/workspaces/:workspace_id/*` behind `scopeMiddleware(RoleOperator |
RoleCompanyAdmin)` with server-verified Kratos session. AI-125 went further:
`/v1/workspaces/:id/facebook-connection{,/connect,/disconnect}` takes a browser
call and fans out to messaging-backend, which is service-token gated east-west.
So "we need an admin BFF" is really "chat-core keeps the role it already has".

- **No companies table in chat_core, and no `/v1/companies`.** Workspaces are
  created only by `POST /internal/workspaces/provision` (service token, ensures a
  company's single DEFAULT workspace). CCS owns companies but exposes them over
  **gRPC only** — a browser cannot reach it without a proxy.
- **Audit already exists**: `workspace_audit_logs` is written on every workspace
  mutation (verified with real rows). What's missing is a browser-facing read
  route, not a storage system.

**chat-core already owns per-workspace templates in 7 places** —
`workspaces.degrade_templates`, `workspaces.disclosure_templates`,
`tier0_rules.templates`, `followup_configs.templates`,
`lead_reminder_configs.templates`, `lead_workflow_configs.template_key`,
`session_summary_configs.summarization_prompt_templates`. AI-96's proposed
`follow_up` template type **collides with `followup_configs`** — decide before
building, or follow-up templates get two owners.

**rag-core auth is broken in a way that outlives the KB editor.** KB writes need
`actor_type == "employee"` plus membership in `kb_editor_groups` from a
gateway-JWT `groups` claim; rag-core has **no service actor type at all**, so
chat-core impersonates an employee. `RAG_CORE_AUTH_TOKEN` must be an RS256 JWT
signed by **channel-gateway, which is decommissioned**, and rag-core refuses
`AUTH_MODE=fake` on staging/prod. That means the **live retrieval path**
(`rag_source/http.go` → `/v1/rag/knowledge/retrieve`) is standing on a token
nobody can reissue — not just the admin screen. Also: rag-core keys scope on
`(provider_id, company_id, workspace_id)` while chat-core has no provider concept
and pins `RAG_CORE_PROVIDER_ID` to one static value per deployment.

Corrected 2026-08-28 after the rag-core owner weighed in, and the correction
matters: channel-gateway did **auth + metering**, and metering already moved to
E5, so only auth is left. If internal service-to-service is declared trusted and
chat-core proxies, the token question largely dissolves — chat-core becomes the
single enforcement point and it already holds Kratos + Keto. My first framing
("needs a platform-wide service-token owner") was heavier than the facts warrant.
Two things still need an answer either way: chat-core→ai-agent uses
`X-Service-Token` **today** (so "internal is authless" is a change, not the
status quo), and rag-core forbids `AUTH_MODE=fake` on staging/prod.

**The sharper question is permission-to-knowledge, and rag-core already models
it.** `visibility` is stored per chunk in Milvus; `resolve_reader` decides
entitlement from the request's `reading_for`, *not* from actor type — AI-46 fixed
exactly the bug where `is_employee=True` meant see-everything and a service
answering customer A could receive customer B's `private` chunks.
`may_read_internal` is employee-only. chat-core does pass `reading_for`
(= `channelIdentityRef`, `context_assemble.go`), so the chain is wired end to
end, and omitting it fails closed to public-only. The open gap is per-actor
entitlement on `workspace_id` (AI-12/Keto), and rag-core's own doc cites a
**"Retrieval Visibility/ACL" card that does not exist on the board**.

⚠️ Consequence of ai-agent being stateless: **context assembly — and therefore
the `reading_for` decision — lives in the caller.** Retrieval is chat-core's job,
generation is ai-agent's (ai-agent references rag-core in zero files). With one
caller this is invisible; a second caller that omits `reading_for` degrades
answers silently, and one that sends the wrong person reopens the AI-46 leak.
Decide shared-library vs move-into-ai-agent vs contract-test before a second
caller exists.

Brief with options and recommendations:
https://claude.ai/code/artifact/68fea22b-9cf8-476a-a90a-71db6bc91b8a

Related: [[ai-platform-deploy-gating]], [[ai-agent-stateless]],
[[ai-merge-topology-risk]], [[ai-board-in-review-means-merged-unverified]]
