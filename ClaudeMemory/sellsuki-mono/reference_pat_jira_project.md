---
name: reference-pat-jira-project
description: Patona Jira project key is PAT — board 71 at sellsuki.atlassian.net
metadata: 
  node_type: memory
  type: reference
  originSessionId: 56602b5c-05ea-4d80-b7ba-3129b2ee78d0
---

# Patona Jira Project

- Project key: **PAT**
- Board: https://sellsuki.atlassian.net/jira/software/projects/PAT/boards/71
- Cloud ID: `50dc7e4a-0539-4dcf-9c33-5481a395a5ab`

## Company Management Epics created 2026-06-18

| Key | Summary |
|-----|---------|
| PAT-2448 | Company CRUD — Create, Edit & Deactivate |
| PAT-2449 | Company Member Management — Invite, Roles & Removal |
| PAT-2450 | Company List — Search, Filter & Pagination |
| PAT-2451 | Company Plan & Quota Dashboard |
| PAT-2452 | Company & Store Settings — Profile, Logo, Store CRUD, Default Store & Company Defaults |

**How to apply:** Use project key PAT when creating/searching Patona tickets. Relates to [[project_qms_ui]] which is also on PAT board.

## MCP gotcha — Epic queries overflow token limit

`searchJiraIssuesUsingJql` always returns the `description` field even when you request only `summary`+`status`. Epics have long descriptions, so a broad query like `project=PAT AND issuetype=Epic` blows the tool-result token limit every time. **Workaround:** delegate the query to a sub-agent (separate context) and have it return only a summary table, OR query narrow by explicit keys. Hit this repeatedly during the 2026-06-22 epic-index refresh.

## Live epic clusters (source of truth: `.claude/knowledge/epic-index.md`)

As of 2026-06-22 the real Jira has large clusters the old epic-index didn't track: **OMS 2.0** (PAT-2238–2247, 2291, 2294, 2361) and **SukiPay 2.0** (PAT-2056–2059, 2064–2069). PAT-2238 cluster IS the roadmap's "Omni-Channel Order Aggregation" (link gap, not coverage gap). PAT-2060–2063 don't exist in Jira (user: ignore). All PAT epics are "To Do".
