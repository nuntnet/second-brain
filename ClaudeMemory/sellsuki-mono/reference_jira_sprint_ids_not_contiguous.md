---
name: jira-sprint-ids-not-contiguous
description: "Sprint ids on sellsuki Jira are global across boards, not per-board — guessing the next id silently files cards into another team's sprint"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 270e02aa-6ae1-461a-ada0-6fb19a8e9e87
  modified: 2026-08-25T11:19:50.712Z
---

Sprint ids in `customfield_10020` are allocated **globally across every board**, not
per board. On the AI board (254) the ids run 3133–3144 for AI Sprint 1–12, then
**jump**: AI Sprint 13/14/15 are **3148/3149/3150**. The gap (3145/3146/3147)
belongs to other boards — 3146 = `LIN Sprint 62` (board 62), 3147 = `DES Sprint 32`
(board 52, active).

**Why this bites:** `editJiraIssue` with a wrong-but-existing sprint id **succeeds
silently** — it files the card into the other team's sprint. On 2026-08-25 that put
17 AI cards into LIN Sprint 62 and DES Sprint 32 before a read-back caught it. JQL
does not help: `sprint in (4000)` and `sprint = "AI Sprint 12"` both return 0 rows
without erroring, so neither id nor name can be validated through the Jira MCP.

**How to get the real ids:** the Jira MCP has no agile-board tool. Use a logged-in
browser tab (Claude in Chrome) on any Jira page and call the agile REST API from the
page context:

```js
await fetch('/rest/agile/1.0/board/254/sprint?maxResults=50&state=future,active',
  {headers:{Accept:'application/json'}})
  .then(r=>r.json()).then(d=>d.values.map(s=>`${s.id} ${s.state} ${s.name}`).join('\n'))
```

**How to apply:** bulk-move is one call per sprint and far cheaper than per-issue
`editJiraIssue` (whose response returns the entire issue description):

```js
await fetch(`/rest/agile/1.0/sprint/${sprintId}/issue`,
  {method:'POST', headers:{'Content-Type':'application/json'},
   body: JSON.stringify({issues:["AI-1","AI-2"]})})   // 204 = OK
```

**Always read back** with `/rest/agile/1.0/sprint/{id}/issue` after writing, and
check the foreign sprints too — a wrong write leaves no error to notice.

Related: [[jira-mcp-search-quirks]], [[jira-editissue-adf-breakage]], [[pat-board-sprints]]
