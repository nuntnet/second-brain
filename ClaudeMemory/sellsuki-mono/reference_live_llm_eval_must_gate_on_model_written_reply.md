---
name: reference_live_llm_eval_must_gate_on_model_written_reply
description: ai-agent live evals through GenerateReply — degrade/escalate bodies are canned per-language templates and pass language or format checks; also TryChat+MessageRef is refused
metadata:
  node_type: memory
  type: reference
  originSessionId: cc764abd-0761-40dd-bf4e-307d5fe2f5d9
  modified: 2026-11-27T11:26:26.718Z
---

When a live eval drives `agent.GenerateReply`, a provider failure returns `Decision=degrade` with a canned template in the request's language. A language or format judge then scores it as a pass even though no model answered. On AI-514, a dead endpoint scored 13/13 until the gate required `Decision==reply` (or `escalate && ModelHandover`).

Also: `GenerateReply` refuses `TryChat: true` together with a `MessageRef` (AI-386). The AI-514 eval's first draft crashed on that and nobody noticed, because the test skips without a key.

**How to apply:** make any live-model eval count only model-written replies. Run it once against a dead `LLM_BASE_URL` with a dummy key: it must FAIL, which proves it reaches the call and is not passing on templates. Related: [[reference_ai_agent_persona_layer_budget_drops_document_checks]]
