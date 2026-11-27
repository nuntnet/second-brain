---
name: reference_ai_agent_persona_layer_budget_drops_document_checks
description: "ai-agent — any text added to the system head (language rule, identity, envelope) silently drops AI-298's completeness check / question checklist on the Gemini document path"
metadata:
  node_type: memory
  type: reference
  originSessionId: cc764abd-0761-40dd-bf4e-307d5fe2f5d9
  modified: 2026-11-27T11:26:22.471Z
---

In sellsuki-ai-agent, everything prepended to `cfg.SystemPrompt` (reply_language.SystemConstraint, identity, live envelope) counts against the persona layer budget (tokens = runes/4). `withDocumentCompleteness` and `boundedDocumentQuestionChecklist` only add their text if the layer has room, and they drop it **silently** when it doesn't. The suite stays green.

Found on AI-514 (2026-11-27). A +76-token draft lost the checklist. A +32-token draft lost the completeness check for a ~750-char Thai persona, which had ~22 tokens of room on main. The fix was to keep the rule at the old length and move the detail into a user-role note before the question, which sits outside the budget.

**How to apply:** for any head or persona-layer prompt change, diff the system text for main vs branch using a realistic persona (500–2,000 chars) on gemini with KB snippets. Prefer per-turn user-role notes (the interest block, the AI-514 language reminder) for added instructions. A second system message would turn the Gemini document cache off. Related: [[reference_live_llm_eval_must_gate_on_model_written_reply]]
