---
name: reference_chat_core_local_inbound_forward_for_live_tests
description: "How to push a real customer message through chat-core's reply pipeline on local without Facebook/LINE or a browser: POST /internal/workspaces/{ws}/messages with X-Service-Token = runtime-keys `chat`; read the reply from `messages`, the stage from `decision_traces`, escalations from `fallback_cases`; harness qa/kb2-2026-10-06/kb2_live_test.py"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-07T03:20:23.243Z
---

**The messaging-backend → chat-core forward route works by hand on local** (contract:
`backend/sellsuki-chat-core/docs/inbound-forward-contract.md`):

```
POST http://127.0.0.1:8099/internal/workspaces/<workspace_id>/messages
X-Service-Token: <runtime-keys.json "chat">   (~/.config/sellsuki/kb-oauth/runtime-keys.json; chat-core SERVICE_TOKEN = keys.chat)
{"external_message_id":"m_<unique>","channel_identity_ref":"facebook:<peer>","channel_conversation_id":"<uuid>","channel":"facebook","body":"..."}
```
Returns 200 at once; the pipeline runs async (1–10 s). Read back from `chat_core` on `sellsuki_mono-postgres-1`:
- `messages` (by `channel_identity_ref` → `chat_session_id`): the AI row has `sender_type='ai'`, `model` (`faq-kb-entry` for a
  verbatim FAQ answer), `tier` (`tier1.5` = FAQ stage), `delivery_status='failed'` with `messaging_client: unexpected status 404
  from send` — expected locally, no page behind the conversation; the body is stored before sending.
- `decision_traces` (by `chat_session_id`): `path` (`faq` | `agent` | `tier0`), `decision` (`faq_exact` | `reply`), `kb_chunks`.
- A low-confidence turn stores NO ai message and NO trace: it writes `fallback_cases` (reason, score), flips
  `chat_sessions.human_mode=true`, logs `outcome":"escalated_to_human"`, and (AI-295) adds a `kb_unanswered_questions` row
  `live_low_confidence`. Every later message in that session gets no AI turn.
- The first AI reply of a session is prefixed with the AI-disclosure line (`chat_sessions.disclosure_sent`).

Workspace `5fcfb2df-…` ("AI-284 Local Smoke"): `confidence_threshold=0.6`, `model=gpt-4o-mini`, **`system_prompt` is empty**, so
most non-FAQ questions escalate on local; set a persona before judging answer quality.
Harness: `qa/kb2-2026-10-06/kb2_live_test.py` (one conversation per case, context turn first; output compatible with
`grade_kb2.py` / `compare_models.py`). Related: [[reference_local_kb_rag_db_and_milvus]], [[project_kb100_eval_state_and_rag_direction]].
