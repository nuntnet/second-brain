---
name: project-ai-mvp-integration
description: "AI chat MVP runs end-to-end locally — integration branches, the inbound forward contract, and the remaining blockers before a real Facebook test"
metadata: 
  node_type: memory
  type: project
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-14T09:34:55.548Z
---

The AI Chat Assistant Platform reached a working local end-to-end path on 2026-08-08. A signed Facebook webhook now flows: signature verify → replay/rate-limit → normalize → **forward to chat-core** → session → Tier 0 keyword match → reply stored → outbound send, stopping only at `token_unavailable` (no real FB page token), which is the correct stop for a credential-free test.

**Branches (nothing merged to main, no MRs):**
- chat-core: `integration/ai-chat-mvp` (union merge of both S6/S7 lines) → `feature/mvp-cross-service-wiring` (RAG client + outbound messaging client + AI-11 seam consolidation) → `fix/inbound-forward-contract`
- messaging-backend: `fix/fb-stack-review-fixes` → `fix/inbound-forward-contract`
- monorepo: `chore/ai-mvp-local-run` (Caddy hosts, ports, `Procfile.ai-mvp`, `make dev-ai-mvp`, `docs/ai-mvp-local-run.md`, `scripts/smoke/fb-webhook.sh`)

**Inbound forward contract** (documented in both repos: `chat-core/docs/inbound-forward-contract.md`, `messaging-backend/docs/chat-core-forward-contract.md`): `POST {CHAT_CORE_FORWARD_URL}/internal/workspaces/{workspace_id}/messages` with `X-Service-Token`, body `{external_message_id (Meta's mid, NOT the per-normalize UUID), channel_identity_ref (the peer), channel_conversation_id (messaging-backend's own cc.ID), channel, body}`. The two ids are deliberately **separate fields** — chat-core stores the conversation id on the session (migration 0043) so the outbound send can address it, without overloading `ChannelIdentityRef` (whose meaning is the customer's channel identity).

**Test without credentials:** seed a Tier 0 keyword rule — a Tier 0 hit answers from a template with zero LLM calls and no RAG, so the whole transport path is exercisable with no LLM key and no Milvus.

**Closed 2026-08-08** on `feature/mvp-media-and-secret-durability` (both repos): secret durability — a Vault dev container is now in docker-compose, `RegisterApp` **refuses** (`503 secret_store_ephemeral`) when a durable registry is wired to an ephemeral secret store (opt-out via `CHAT_ALLOW_EPHEMERAL_SECRET_STORE`), and "secret gone" is now `ErrFBAppSecretUnresolvable`/`503` instead of masquerading as `404 fb_app_not_found`; attachments — the forward payload and chat-core's inbound DTO carry an attachment (body OR attachment required, migration 0044). Caught along the way: a tier0 `.*` pattern rule would have fired a canned text reply at every incoming photo, because `matchesPattern` didn't reject an empty body.

**REAL FACEBOOK END-TO-END WORKS (2026-08-08).** A message sent from Messenger to the pilot Page gets an AI reply delivered back, live. Pilot config: app `235756003289739` ("Sellsuki-New", Development mode, Business type) · Page `268502970215081` ("Marketing Space") · company `company-fb-pilot` · workspace `c11511cc-ecb5-4579-95b0-c5421be87e84` · public webhook `https://ai-chat-local-api.bearyweb.com/webhook/fb/235756003289739` (a hostname added to the pre-existing cloudflared named tunnel `bola-local` on zone `bearyweb.com`; config at `~/.cloudflared/config.yml`, backup `.bak`) · page token exchanged, `token_status=healthy`. Verified: signature verify → replay/rate-limit → normalize → forward → session → Tier 0 → reply stored → delivered to Messenger; the first AI message correctly carries AI-57's bot disclosure.

**Debugging lesson that cost an hour:** the App ID was transcribed from a screenshot and one digit was misread (`...005...` vs the real `...003...`). Every failure downstream was Meta's generic `101 Invalid Client ID`, which was misdiagnosed as a bad App Secret — the user reset their app secret needlessly. Also learned: `GET graph.facebook.com/v{v}/{id}` **cannot** be used to prove an app id is valid (a real app id returns "does not exist" without a token, while an unrelated object id returns 104). **Never transcribe an ID from an image — take it from the URL or ask for it as text.**

**⚠️ Before wiring RAG, fix the ACL — it currently leaks across customers.** Found during AI-63 and verified directly in source: `rag-core/src/use_case/internal/knowledge_visibility.py:30-31` starts with `if is_employee: return list(hits)`, short-circuiting the whole public/private + `owner_customer_id` check below it. chat-core's client (`src/repository/rag_source/`) sends no `requester_customer_id` and **drops the `visibility` field** when mapping the response, so it can neither be filtered by rag-core nor re-check on its own. Wire it as-is and a customer's question can surface another customer's `private` chunk or an `internal` doc. Deliberately not fixed in AI-63 (the ACL model is AI-5/AI-46's call and guessing is harmful in both directions); pinned by `src/repository/rag_source/acl_boundary_test.go`. Full write-up + the three decisions needed is a comment on AI-46.

**RAG is deliberately not wired (decision 2026-08-11).** The RAG team is reworking their POC API to be **per-company**, while our `rag-core` work (AI-38/39/41/43/44) is **per-workspace** — connecting now would mean redoing the contract twice, so AI-46 (retrieval visibility/ACL + PDPA erasure) is deferred until their API lands. The chat-core side is already built behind a port (`src/repository/rag_source/`), so the swap is one adapter. Consequence while unwired: `hasKBGrounding` is hard-false, so AI-57's guardrail rewrites/blocks any numeric claim — expect behaviour to change and re-test once RAG connects.

**RE-VERIFIED LIVE 2026-08-14** (items 1 and 2 below are now CLOSED). The whole
MVP path was running in `.overmind-ai-mvp.sock` and re-proved end-to-end:
Tier 0 via `scripts/smoke/chat-core-inbound.sh -k local-dev-token`, the full
signed chain via `scripts/smoke/fb-webhook.sh`, and **Tier 2 with a real
OpenRouter key** (`LLM_API_KEY` now lives in root `.env.ai-mvp`, gitignored,
sourced per-line by `Procfile.ai-mvp`) returning a 3.5k-char Thai reply with
`ai_error: null`. `file-service` + `role-perm` are now in `Procfile.ai-mvp`,
and `ai-agent` (8101) is wired there too after the E3 extraction.
The FB pilot survives: page `268502970215081` → workspace
`c11511cc-ecb5-4579-95b0-c5421be87e84`, `token_status=healthy`, and the
cloudflared hostname `ai-chat-local-api.bearyweb.com` still routes to 8100
(a bad `hub.verify_token` correctly returns 403) — so a real Messenger test is
one message away, no re-onboarding needed.
Gotchas hit: the smoke scripts read the token from
`backend/sellsuki-chat-core/.env`, which does NOT match the Procfile's
`SERVICE_TOKEN=local-dev-token` — pass `-k local-dev-token` or you get
`401 invalid_service_token`. And **kratos-ui had died while overmind still
reported it "running"** (port 4455 not listening → `accounts.sellsuki.local`
502 → no admin login); `OVERMIND_SOCKET=.overmind-ai-mvp.sock overmind restart kratos-ui`
fixed it. Trust `lsof`, not `overmind status`.

**Still blocking a fuller test:**
1. ~~`CHAT_FILE_SERVICE_BASE_URL` unset~~ — CLOSED, see above.
2. ~~`LLM_API_KEY` unset~~ — CLOSED, see above.
3. Dev-mode Vault is inmem, so recreating the container still loses secrets — but it's now loud and recoverable with `rotate-secret`.
4. chat-core can't mint a conversation id itself (`GetOrCreateConversation` needs a `channel_binding` it has no model for) — fine while every conversation arrives via this forward.

Non-text messages today are **stored, not answered** — guardrails and tier0 both see an empty body and don't fire; deciding AI behaviour for media is AI-4's call and was deliberately not invented here.

See [[project-ai-sprint234-autonomous-run]] for the full card/branch map and [[gorm-pgx-libpq-automigrate]] for the restart bug found along the way.
