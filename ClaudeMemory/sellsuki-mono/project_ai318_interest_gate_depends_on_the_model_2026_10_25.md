---
name: project-ai318-interest-gate-depends-on-the-model-2026-10-25
description: First live measurement of AI-318 interest classification on the 76-row Thai set: gemini-2.5-flash 0.708/0.810 (fails the 95/85 gate), gpt-4o-mini 0.952/0.952 (passes); topics help one model and hurt the other; set is the code author's own, one run each
metadata:
  node_type: project
  type: project
---

Measured 2026-10-25 through the running local ai-agent: `POST :8101/internal/v1/workspaces/<ws>/reply` with header `X-Service-Token: local-dev-token`, body = persona + `interest_context.kb_topics` from `backend/sellsuki-ai-agent/src/use_case/agent/testdata/interest_eval_th.json` (script and raw results in `docs/cards/evidence/AI-318/`). Results (precision/recall): gemini-2.5-flash + topics 0.708/0.810; gpt-4o-mini + topics 0.952/0.952; gemini no topics 0.864/0.905; gpt-4o-mini no topics 0.833/0.952. The local workspaces (w2, W3, Test Insurance Co) are configured with gpt-4o-mini; the KB answer model is gemini-2.5-flash ([[feedback_gemini_25_flash_is_the_kb_answer_model]]). Failure shape of gemini: contentless messages ('?', '..', '👍', 'ได้') classed `interest` with a reason copied from the KB topics.
**Why:** the gate depends on the model, so the card has to say which model classifies; tuning the prompt on this same set would overfit (labels are the code author's, gate still 'awaiting PO confirmation'). Inbox D-038 (written before the extra runs; the ledger holds the table).
**How to apply:** before claiming AC1, re-run `eval.py <model> [notopics]` against whatever model the workspace really uses, on a held-out PO-confirmed set; do not tune on the 76 rows. The flag source is CCS :8085 (not running locally), so the flag path cannot be switched on locally.
