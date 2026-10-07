---
name: feedback_gemini_25_flash_is_the_kb_answer_model
description: "PO decision 2026-10-07: model-selection phase is CLOSED — google/gemini-2.5-flash is the only answer model for the KB/whole-document path from now on; claude-sonnet-4-5 is too expensive for production and for experiments; optimise code/prompt/retrieval with gemini until 100/100 on the frozen set"
metadata:
  type: feedback
---

**Decision (user, 2026-10-07 15:40):** "หลังจากนี้เรา gemini-2.5-flash เท่านั้น ปิด phase เรื่องการเลือก model" — whole-document run
on the frozen 100 (gpt-4o judge): sonnet 92 ($3.74/100), gemini-2.5-flash 88 ($0.21/100, p50 1.5 s), glm-4.6 81, gpt-4.1-mini 81.
Sonnet is out for production AND for test runs (cost). Judge for experiments: gpt-4o.

**Why:** cost/latency per message; 88 vs 92 is within what retrieval/prompt work can close.

**How to apply:** every new run of `step1_wholedoc.py` / Playground harness uses `--direct --model google/gemini-2.5-flash`
until the model is in the ai-platform-kit catalog (`llmclient/catalog.go`, slug `google/gemini-2.5-flash`) and the agent
can route to it; do not propose or benchmark other models unless the user reopens the question. The goal is 100/100 on
`qa/kb100-2026-10-01/questions.json` through the PRODUCT path (not the oracle-routed script) — plan step 3 in
`qa/kb100-2026-10-05/plan-kb-quality-2026-10-06.md`. Related: [[project_kb100_eval_state_and_rag_direction]].
