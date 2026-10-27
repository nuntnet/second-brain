---
name: reference_ai_chat_dev_order_reconcile_2026_10_27
description: "2026-10-27 reconcile of all 350 AI Jira cards vs roadmap → artifact with 5 waves; key state: staging chat module OFF since 10-27 11:40 (messaging 077ebd3), 10 staging bugs AI-335..346 and AI-347..350 sit in no sprint, Sprint 21-24 plan still valid, AI-291 gates Sprint 24"
metadata:
  node_type: memory
  type: reference
  originSessionId: 80111067-f1d1-45c3-948c-127f780fabcb
  modified: 2026-10-27T11:21:17.617Z
---

Artifact (private): https://claude.ai/artifact/VRvko7XAYxwZkaGujyUPqQ — ลำดับ develop AI Chat 5 ระลอก (0 ปลดบล็อก ops · 1 บั๊ก staging 26 ต.ค. + ปิด Sprint 21 · 2 = Sprint 22 + แทรก AI-347/350 · 3 = Sprint 23 + AI-112/230 · 4 = Sprint 24 รอ AI-291) + ตาราง epic + drift + Decision Inbox. Source files: scratchpad `ai-chat-dev-order.html`, Jira TSV `ai_full.tsv` (key/type/status/parent/sprint).

Board facts worth not re-deriving (as of 2026-10-27): 350 cards = 160 To Do / 109 In Review / 25 In Progress / 56 Done; parent via field `parent`, sprint via `customfield_10020` (search tool must be asked for them explicitly); Sprints 1–20 all still `future`, never closed; epics AI-1/2/4/5/8 To Do with children done; Sprint 21=3301 (218,314,315,316,318,325,326,327) 22=3302 (317,319,312,213,331–334) 23=3303 (313,322,320,329,297) 24=3304 (321,323,328).

**Why:** the ordering question keeps coming back and the board cannot answer it (In Review ≠ verified, old sprints never closed). The artifact is the durable copy; the biggest blocker is not a feature but `CHAT_MODULE_ENABLED=false` on staging and prod (AI-216 commits 077ebd3 / dca6851, secrets omitted so OTP can boot).
**How to apply:** before re-planning, check (1) whether the chat module is back on, (2) whether AI-335..346 and AI-347..350 got a sprint, (3) whether AI-291 merged. See [[project_wave1_playbook_goal_lead_kb_state_2026_10_25]], [[project_playbook_goal_lead_kb_cards_2026_10_24]], [[project_ai_chat_staging_permission_state_2026_10_27]], [[reference_ai_board_in_review_means_merged_unverified]].

**PO decision 2026-10-27 (same session):** AI Sprint 21 (3301) is "current" even though Jira has no active sprint (all 24 are `future`; starting/closing a sprint is the PO's click, not MCP). PO agreed Sprint 16–20 leftovers (66 cards) should not stay there: ~10 pilot-path cards move into 21–24 (AI-240, 107, 55, 183, 230/112/232, 111, 114), ~30 platform-track cards go to backlog with a `platform-track` label (E1b, E6, E11, E9, E12, admin extras), In Review/In Progress ones get a verification sweep not a move, and 5 spikes (AI-145, 151, 221, 77, 118) wait for the PO to confirm closing. **Applied 2026-10-27 18:33–18:41 ICT** (evidence `docs/cards/evidence/SPRINT-CLEANUP-2026-10-27.md`): Sprint 21 now 31 cards (8 original + AI-240, 55, 9 `verify-sweep` In Review, 12 bugs AI-335..346), Sprint 22 = 14 (+107, 225, 81, 82, 87, 90, 347-350), Sprint 23 = 9 (+183, 230, 112, 232), Sprint 24 = 6 (+111, 114, 244); 36 platform-track cards out of any sprint; AI-221/174/145/151 closed Done with comments (145 = Discord cut). Verified by JQL, 0 mismatch. Still the PO's click: start Sprint 21, complete Sprints 1-20. `editJiraIssue` with `customfield_10020: <sprint id>` moves a card; `null` removes it; parallel edits of 4-18 per message worked without rate-limit errors.
