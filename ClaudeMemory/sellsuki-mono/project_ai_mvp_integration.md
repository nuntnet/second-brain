---
name: project-ai-mvp-integration
description: "AI chat MVP runs end-to-end locally — integration branches, the inbound forward contract, and the remaining blockers before a real Facebook test"
metadata: 
  node_type: memory
  type: project
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-08T10:16:52.217Z
---

The AI Chat Assistant Platform reached a working local end-to-end path on 2026-08-08. A signed Facebook webhook now flows: signature verify → replay/rate-limit → normalize → **forward to chat-core** → session → Tier 0 keyword match → reply stored → outbound send, stopping only at `token_unavailable` (no real FB page token), which is the correct stop for a credential-free test.

**Branches (nothing merged to main, no MRs):**
- chat-core: `integration/ai-chat-mvp` (union merge of both S6/S7 lines) → `feature/mvp-cross-service-wiring` (RAG client + outbound messaging client + AI-11 seam consolidation) → `fix/inbound-forward-contract`
- messaging-backend: `fix/fb-stack-review-fixes` → `fix/inbound-forward-contract`
- monorepo: `chore/ai-mvp-local-run` (Caddy hosts, ports, `Procfile.ai-mvp`, `make dev-ai-mvp`, `docs/ai-mvp-local-run.md`, `scripts/smoke/fb-webhook.sh`)

**Inbound forward contract** (documented in both repos: `chat-core/docs/inbound-forward-contract.md`, `messaging-backend/docs/chat-core-forward-contract.md`): `POST {CHAT_CORE_FORWARD_URL}/internal/workspaces/{workspace_id}/messages` with `X-Service-Token`, body `{external_message_id (Meta's mid, NOT the per-normalize UUID), channel_identity_ref (the peer), channel_conversation_id (messaging-backend's own cc.ID), channel, body}`. The two ids are deliberately **separate fields** — chat-core stores the conversation id on the session (migration 0043) so the outbound send can address it, without overloading `ChannelIdentityRef` (whose meaning is the customer's channel identity).

**Test without credentials:** seed a Tier 0 keyword rule — a Tier 0 hit answers from a template with zero LLM calls and no RAG, so the whole transport path is exercisable with no LLM key and no Milvus.

**Blockers before a real Facebook test** (beyond obtaining a page token):
1. `VAULT_ADDR` unset ⇒ the secret store is in-memory while `fb_app` rows live in Postgres, so **every messaging-backend restart loses the app secret** and all webhooks 404 `fb_app_not_found`; the row must be deleted and re-registered (rotate-secret can't recover it).
2. Attachments don't flow — chat-core's inbound route models a text `body` only, so media messages are refused with `unsupported_content` before any HTTP call. Fix is additive (`attachment_ref` + relax body-required).
3. chat-core can't mint a conversation id itself (`GetOrCreateConversation` needs a `channel_binding` it has no model for) — fine while every conversation arrives via this forward.

See [[project-ai-sprint234-autonomous-run]] for the full card/branch map and [[gorm-pgx-libpq-automigrate]] for the restart bug found along the way.
