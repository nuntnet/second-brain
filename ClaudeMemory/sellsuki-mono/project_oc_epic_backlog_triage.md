---
name: project_oc_epic_backlog_triage
description: OC2Plus Jira board has ~127 non-Done epics (heavy stale clutter); 26 dead-shell epics closed 2026-06-22; MCP Jira server returns cross-project contaminated/paginated results
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

OC2Plus board (project key `OC`, id 10001, site sellsuki.atlassian.net, cloudId `50dc7e4a-0539-4dcf-9c33-5481a395a5ab`) carries a very large stale-epic backlog.

**Counts (verified live 2026-06-22):** ~127 non-Done epics + ~79 Done = ~206 total. Many are dead shells from 2021–2024 (empty description, release/version/load-test buckets, abandoned ML/ad concepts). Old backlog epics do NOT show on the team's board view (board filter scopes them out) but still exist in the project — reach them via Issues Search `project = OC AND issuetype = Epic`.

**Closed 2026-06-22 (54 epics total → Done).** Transition to Done = id `31`, global. OC statuses: To Do/In Progress/Blocked/Done only — no separate Closed.
- Round 1 (26 safe-tier dead-shells): OC-1739, OC-4045, OC-3866, OC-1571, OC-33, OC-32, OC-36, OC-22, OC-31, OC-101, OC-35, OC-1454, OC-209, OC-1565, OC-18, OC-1981, OC-1999, OC-2005, OC-2001, OC-1966, OC-1729, OC-1750, OC-2237, OC-2006, OC-2185, OC-2057.
- Round 2 (28 more): OC-2224, OC-1740, OC-1741, OC-1742, OC-1743, OC-1745, OC-1746, OC-1747, OC-1748, OC-1749 (release buckets 1.7→1.14 + Posh docker-compose), OC-5, OC-9, OC-27, OC-34, OC-59, OC-1415, OC-1752, OC-1754, OC-1755, OC-1756, OC-1757, OC-1758, OC-1759, OC-1982, OC-2088, OC-2155, OC-2156, OC-2190.

**Deliberately NOT closed (user didn't authorize the Ad/ML/Marketplace group):** OC-38, OC-39, OC-41 (FB/Google/Tiktok Ads), OC-44 (Wallet), OC-57 (Call Center), OC-222, OC-223 (ML), OC-26 (Link Tracker), OC-1753 (Shopee/Lazada), OC-1751 (FB language analysis), OC-1760, OC-1761 (FB Ads pipeline/label) — 12 epics, candidate-close pending OK.

**KEEP (active/recent, do not touch):** OC-4223, OC-4224 (Bola x OC2Plus, part of OC-4200 P0 chain), OC-4130 (Posh docker image, recent), OC-2241 (Campaign Product Condition). Plus the Tier-3 (13) + Archive (26) groups from the kill-list artifact still un-actioned.

**⚠️ MCP Jira bug (recurring):** the Atlassian MCP server returns cross-project contaminated rows under load (PAT-xxxx issues appear in OC queries) and truncates/mis-paginates. `searchJiraIssuesUsingJql` ignores the `fields` restriction and returns huge payloads → use jq on the saved tool-result file. Don't trust agent-aggregated Jira counts; verify key lists with direct `key in (...)` JQL. The kill-list in an earlier round wrongly claimed OC-4200/4225/4283 don't exist — they DO (they're real epics on page 2). See [[reference_oc2plus_jira_project]].
