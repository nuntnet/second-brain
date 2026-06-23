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

`searchJiraIssuesUsingJql` always returns the `description` field even when you request only `summary`+`status` (markdown format makes it worse). Epics have long descriptions, so a broad query like `project=PAT AND issuetype=Epic` blows the tool-result token limit every time. **Best workaround (confirmed 2026-06-23):** the overflowing result auto-saves to a file under `tool-results/` — pipe that file through `jq` to extract only the fields you want, e.g. `jq -r '.issues.nodes[] | "\(.key)\t\(.fields.status.name)\t\(.fields.summary)"' <file>`. Same trick works for `issuelinks`. No sub-agent needed.

## Live epic clusters (source of truth: `.claude/knowledge/epic-index.md`)

As of 2026-06-23 the real Jira has **75 epics** total, only 3 Done, the rest "To Do" — **no epic is ever marked In Progress** (delivery is tracked at story/sprint level, so epic status ≠ real state). ~35 are legacy clutter (OMS 1.x, CCS 1.x, i18n, AMS, Akita 3.1.x, MVP, QMS) that will never move — triage candidates like the OC board.

Real active clusters: **OMS 2.0** (PAT-2238–2247, 2291, 2294, 2361, +2120/2122 OpenAPI/MCP) and **SukiPay 2.0** (PAT-2056–2069, +2052/2121/2440/2054/1960). PAT-2238 cluster IS the roadmap's "Omni-Channel Order Aggregation" (link gap, not coverage gap). **PAT-2060 DOES exist** (SukiPay Foundation Services — Audit/Webhook/File) — corrects earlier note; 2061–2063 still absent. epic-index.md still lists PAT-2060 as "not found" → drift, needs refresh.

## OMS 2.0 epics have no dependency links

Jira issuelinks between OMS 2.0 epics are only "relates to" + children — there are **no blocks/blocked-by links**, so the cluster has no ordering in Jira. Sequencing must be **derived from architecture** ([[oms2-architecture]] skill: foundation PAT-2238+2239 → WF-A slices POS/ONLINE → integration payment/shipmunk/akita → WF-B marketplace → platform). Demo spine = PAT-2238 → 2239 → Slice 1 POS (S100). Cross-cluster lock: OMS PAT-2244 ↔ SukiPay PAT-2057 must be same-sprint.
