---
name: reference_local_chat_core_api_drive_recipe_and_consent_gap
description: "How to drive chat-core's real HTTP API on the local ai-mvp stack as a verifier (Kratos cookie via :5191, internal X-Service-Token, VL3 test data, cleanup) plus the known local blockers (no consent record, no Google client, no BOLA, file-service permission)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 51033af8-906c-46a8-a663-9741ee8d35ab
  modified: 2026-10-29T15:14:55.027Z
---

Learned 2026-10-29 verifying Sprint 23 (docs/cards/sweeps/2026-10-29-local-verify-s23.md).

- Sandboxed Bash cannot reach localhost or the overmind socket; probes need `dangerouslyDisableSandbox`. `rtk proxy curl` is needed to read JSON (rtk collapses bodies to a schema).
- Console API: log in to Kratos at 127.0.0.1:4433 with python urllib (identity from `.claude/verify/.local/credentials.json`; token is the `sellsuki_session` Set-Cookie), then call `localhost:5191/v1/workspaces/<ws>/...` with `Cookie: sellsuki_session=<token>`. A fresh login does not break the older cookie. For the browser pane set the same cookie with `document.cookie` on localhost:5191. Inbox deep link is `/w/<ws>?session=<id>` (not `/conversations/<id>`); first paint takes 5-10 s.
- Internal routes use header `X-Service-Token: local-dev-token` (not Authorization Bearer). A simulated customer message: `POST :8099/internal/workspaces/<ws>/messages` (creates session + case; triggers an ai-agent attempt).
- Seed identity is Company Owner of company 85fa00dd… / workspace 2d78a4b0…; it cannot reach any second company, so "2 company" ACs only prove denial (403/404). Other-company workspace example: 5fcfb2df….
- First-bind PUTs for sheet-export/form-watch configs need `"version":0`. Cases need `subject:{"type":"self"}`.
- Local blockers that make ACs unverifiable, not failed: consent record can never be created (AI-77 gap, consent service not running) so reminder/lead-data PII writes 403; no Google client (503 SHEET_*_NOT_CONFIGURED); BOLA not running (group verifier 503/422); seed identity lacks file-service upload permission (attachment upload 500).
- Cleanup: delete only VL3-prefixed rows by id via `docker exec sellsuki_mono-postgres-1 psql -U postgres -d chat_core`, in one transaction (leads/lead_data/audits/stage_events, customer_facts by channel_identity_ref, case_audit_logs, messages, chat_sessions, cases, decision_traces, config tables). chat-core local/stack is rebuilt by air whenever another session commits, so :8099 drops for ~30 s; wait, don't touch it.
