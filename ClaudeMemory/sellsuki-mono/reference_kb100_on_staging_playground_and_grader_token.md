---
name: reference_kb100_on_staging_playground_and_grader_token
description: "Run the frozen KB100 on staging from the console pane (in-page fetch loop, retry 'unavailable' serially), grade_llm.py judge token in runtime-keys.json is rejected → scratch copy with local-dev-token; customer KB is text-embedding-3-small so D-057 (gemini-embedding) only concerns internal Outline/Drive/Gmail collections (2026-10-30, U5)"
metadata:
  node_type: memory
  type: reference
  originSessionId: e5a9c0e7-41e9-4dc8-a72d-71495c2ad13f
  modified: 2026-10-30T13:13:58.470Z
---

Used for MVP wave 2 U5 (2026-10-30), results in `docs/cards/sweeps/2026-10-31-mvp-accuracy.md`.

- **Run:** in the logged-in ai-chat-admin.staging-th pane, put the 100 questions in `window.__qs` and loop `fetch(.../v1/workspaces/5266e73a-…/kb/playground)` with 4 workers, storing results in `window.__res`; read back in two halves of 50 (one return is too long). ~10 min for 100 questions.
- **Trap: 4 concurrent requests → ~10% `status:"unavailable"`** (200, empty answer). Retry those serially before grading, or they count as misses.
- **Trap: grader token.** `qa/kb100-2026-10-05/grade_llm.py` reads `~/.config/sellsuki/kb-oauth/runtime-keys.json["chat"]`; local ai-agent rejects it, so every row prints "judge unavailable/malformed". Grade with a scratchpad copy that sets `token = "local-dev-token"` and an absolute ROOT; don't edit the frozen harness.
- **Grader layer-1 quirk:** `needs_clarification` on an unanswerable question counts as `wrong` (only `no_evidence` = abstention). Review those by hand.
- **Embedding fact:** the customer KB ingest path is `text-embedding-3-small`/1536 (rag-core `INGEST_EMBEDDING_MODEL`); the 3072-dim gemini-embedding vectors are only the internal collections. A customer-KB accuracy number does not bear on D-057.
- **Local guardrail probes** via `scenario.py` need Milvus up; otherwise every turn that reaches the model degrades after 5 RAG-503 retries and only keyword-gate results are observable.

Related: [[project_kb100_eval_state_and_rag_direction]], [[reference_staging_kb_playground_fetch_from_console_pane]], [[reference_real_conversation_scenario_harness_lead_goal_step_2026_10_24]]
