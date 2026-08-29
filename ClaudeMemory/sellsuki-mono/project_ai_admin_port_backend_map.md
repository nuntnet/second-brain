---
name: ai-admin-port-backend-map
description: "ai-chat-admin-frontend's 'backend does not exist' port comments understate reality — 3 of 8 have backends; KB is rag-core's and complete; the real blocker is a missing rag-core seam"
metadata:
  node_type: memory
  type: project
---

`apps/admin/src/main.tsx` substitutes ~8 ports with "unavailable" adapters and calls the list "the todo list". **Read it as a lower bound, never as fact.** Verified 2026-08-29 by grepping route registrations across every branch of chat-core and rag-core:

| port | truth |
|---|---|
| `configVersionPort` | backend EXISTS — chat-core `GET /v1/workspaces/:id/config-versions`, `POST .../rollback` (AI-99, MR !41). Adapter missing only. |
| `kbEntryPort` | backend COMPLETE incl. list — **rag-core**, not chat-core: `GET/POST/PUT/DELETE /v1/rag/knowledge/entries` (MR !18). The list route's docstring names "AI-96's KB tab". |
| `leadNotifyPort` | PARTIAL — read maps to `GET /hot-lead-notify-config`; update splits across `PUT /enabled` + `POST/DELETE /destinations`; `listBolaLineGroupTargets` / `testSendNotify` have no north-south route. |
| `kbTemplatePort`, `adsSpendPort`, `quotaPort`, `leadSuggestionPort`, `workspacePort` list/create | genuinely nothing, any branch, either repo. |

**KB is blocked by a SEAM, not an endpoint.** The admin app has no rag-core base URL, HTTP client or Vite proxy entry — it speaks to chat-core and Kratos only. Wiring it means a chat-core BFF route forwarding to rag-core (the pattern every other port follows) — and rag-core has no service actor for that hop. That is a product decision, not wiring. See [[reference_chatcore_is_the_admin_bff]].

**rag-core's KB stack is one linear chain, all inside MR !18:** `main → AI-38 → AI-39 → AI-41 → AI-43 → AI-46`. Each branch contains all the previous, so !18 (AI-46 → main) merges the whole thing. Do not open MRs for the intermediate branches — they are not stranded. All are ~15 commits behind main.

**How to apply:** before building any "missing" AI-platform backend, grep the route registrations on the *tip of the stack*, not on the card's own branch — the AI-43 GAP note was accurate for its own branch and wrong for the stack, because the list endpoint landed later. Corrected in ai-chat-admin-frontend MR !24. Related: [[project_ai_merge_topology_risk]], [[reference_ai_board_in_review_means_merged_unverified]].
