---
name: project_kb100_eval_state_and_rag_direction
description: "AI Chat KB quality — frozen 100-question FWD insurance eval, latest 55/100 (2026-10-06), where the artefacts are, the 5 root causes, and the user's open question whether to drop chunk RAG for document-level/agentic reading"
metadata:
  node_type: memory
  type: project
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-06T11:37:22.474Z
---

**State (2026-10-06):** Codex ran a frozen 100-question eval (FWD insurance KB, 12 Drive files) against
`AskKBPlayground` on local. Scores: 30 (1 Oct) → 25 → 33 → 37 → 47 → 48 → 54/58 → **55** (49 factual + 6
correct abstentions). Same code reruns swing ±3. Artefacts: `qa/kb100-2026-10-01/` (questions.json, sources/,
SHA manifest), `qa/kb100-2026-10-05/*grades*.json` + `*.jsonl` (grades files have a `grades` list under
metadata keys), Codex ledger `docs/cards/AI-291.md` (2,500+ lines), my review
`qa/kb100-2026-10-05/review-2026-10-06-claude.md`.

**Root causes I verified** (details + file:line in the review): table headers split from values by the
800-char chunker (Precious p4 header at char 617, values at 1114+); 90/20 table emitted as HTML `<td>` which
the chunker treats as prose; rag-core context ceiling `hybrid_max_context_tokens=8000` counts UTF-8 BYTES so
Thai gets ~2,600 chars (model receives median 6 of the 15 requested passages); Milvus sparse tokenizer keeps
whole Thai runs as one token (hybrid is dense-only for Thai); questions say Platinum/Bronze, corpus only has
แพลทินัม/บรอนซ์; title-hint routing on `ci50` pulls the FAQ xlsx; live customer path has only 900 RAG tokens
vs 4096 in Playground, so real chat is worse than the measured 55.

**Prose questions already pass 28/31; everything missing is tables** (precious/spreadsheet/price 17 of 48).
29 of 45 failures had the answer's numbers in the evidence but no header/label → model abstained correctly.

**User's direction question (2026-10-06):** asked why Claude reading the files directly answers almost
everything while the RAG path does not, and whether to stop using RAG. My answer: keep retrieval, drop
chunk-RAG for small corpora — route at document level + stuff whole brochure with prompt caching, give the
agent a read-document tool with 2–3 iterations, structured lookup (AI-291) for price tables, stronger model
(gpt-4o / claude-sonnet-4-5 are in the catalog). Offered a cheap decisive experiment: feed the 100 questions
the full source doc (oracle routing via the `source` field) to a stronger model, no chunking. Not yet run.

**Why:** the user is deciding the architecture of the KB answer path; a new session must not restart the
diagnosis or re-propose "more context / better prompt", both already tried and measured by Codex.
**How to apply:** start from the review file and the latest grades; any improvement claim needs a full
frozen-100 rerun (3× preferred), never a spliced smoke. Related: [[reference_chatcore_ragcore_two_hops]],
[[project_ragcore_dual_embedding_paths]], [[reference_local_kb_rag_db_and_milvus]].
