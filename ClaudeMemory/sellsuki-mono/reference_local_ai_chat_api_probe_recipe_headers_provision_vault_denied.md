---
name: reference_local_ai_chat_api_probe_recipe_headers_provision_vault_denied
description: "Probing the local ai-mvp stack by curl — Kratos login via python, chat-core internal provision with local-dev-token, ai-agent /internal/v1/llm needs X-Caller-Service header (feature is header or vocabulary body), Vault secret read is classifier-denied so signed FB events are out of reach"
metadata:
  node_type: memory
  type: reference
  originSessionId: a0a96da3-7a12-4514-9746-54a1a54ee991
  modified: 2026-10-29T15:16:07.607Z
---

Learned 2026-10-29 (local verify of Sprint 21):

- Localhost is blocked inside the sandbox; run curl/psql/go test against the stack with the sandbox off. `rtk` truncates curl/grep/tail output (a series looks "missing" when it is not) — use `rtk proxy curl -o file` and read with python.
- Session: Kratos browser-flow login at 127.0.0.1:4433 with `.claude/verify/.local/credentials.json` (key `kratos`), cookie `sellsuki_session` replayed to chat-core :8099. `GET /v1/me/workspaces` lists grants.
- New test tenant without Keto: `POST /internal/workspaces/provision` or `/internal/workspaces/companies/:id` with `X-Service-Token: local-dev-token` (value in Procfile.ai-mvp). The provisioned company gives the seed identity NO role (CCS grants roles), so only workspaces inside company `85fa00dd…` are usable by the seed user. No delete route: clean up by `delete … where workspace_id in (…)` over every public table with that column in db `chat_core` (container sellsuki_mono-postgres-1), slow (~100 s).
- ai-agent `/internal/v1/llm` and `/embeddings`: service token + header `X-Caller-Service` (+ `X-Feature`); body `workspace_id`, `company_id`, `model`. Real OpenRouter key works locally. QMS is not in the stack (`refused code=unavailable`).
- chat-core DTO gotchas: goal PUT `cta_text` is a `{th:..}` map and `version` must be >=1; fact-schema purpose enum is kb_grounding|lead_signal|enrichment.
- Reading the local Vault app secret for fb_app signing was DENIED by the auto-mode classifier (credential materialization) — signed webhook events cannot be made by an agent; hand that to the user.
- Console cookie recipe: see [[reference_local_ai_chat_console_open_on_localhost_with_seed_identity]]; the sidebar is not in the a11y tree, click by coordinate.
