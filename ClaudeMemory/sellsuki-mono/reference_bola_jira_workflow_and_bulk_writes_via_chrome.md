---
name: reference_bola_jira_workflow_and_bulk_writes_via_chrome
description: "BOLA board (id 89) workflow transition ids, resolutions, sprint ids, and how to bulk-write via logged-in Chrome REST (cheap) — bulk closes need the user's explicit approval first or the auto-mode classifier blocks"
metadata:
  node_type: memory
  type: reference
  originSessionId: 37718d4a-c6c6-4da2-ad71-349c1d7ff383
  modified: 2026-10-12T17:09:45.117Z
---

Verified 2026-10-12/13 on the BOLA board (project BOLA, board 89, cloudId 50dc7e4a-…).

- **Transition ids (global, any→any):** 11 To Do · 21 In Progress · 31 Done · 41 In Analy · 51 Ready for Design · 61 In Design · 71 Ready for Dev · 81 In Progress (DEV) · 91 Ready for QA (SER) · 101 In QA (SER) · 111 Blocked. No "code review" status.
- **Resolutions exist:** Done, Won't Do, Duplicate, Cannot Reproduce. Close = transition 31, then `PUT /rest/api/3/issue/KEY {fields:{resolution:{name:"Won't Do"}}}` (works, 204).
- **Bulk path that works:** `claude-in-chrome` tab on sellsuki.atlassian.net, then `fetch()` in `javascript_tool`: comment `POST /rest/api/3/issue/KEY/comment` (ADF body), transitions, `POST /rest/agile/1.0/sprint/{id}/issue` (≤50), `POST /rest/agile/1.0/backlog/issue`. Responses are tiny, unlike MCP editJiraIssue which returns the whole description. Keep each call to ≤~14 cards or CDP times out at 45s (the script still finishes — verify by JQL).
- **Sprint ids (board 89):** 766=S1, 1098=S2 (both still `active`, overdue since June — need a human to complete), 1099=S3, 1459=S4, 2915=S5, 2916=S6, 2920=S7, 2921..2928 = S8..S15 (empty, reusable). Set on 2026-10-12: S8=2921 close-out engineering, S9=2922 QA sweep, S10=2923 next. Sub-tasks follow their parent's sprint (BOLA-72 follows 49).
- **Auto-mode classifier blocked a 65-card bulk close** ("External System Writes") until the user approved the closure list in chat; after approval the same calls passed. Present the closure list first, don't retry around the block.
- glab inside the sandbox fails TLS (x509 -26276) → run read-only glab with the sandbox disabled.

Related: [[reference_jira_board_sweep_via_saved_results]], [[reference_jira_sprint_ids_not_contiguous]], [[reference_bola_jira_project]].
