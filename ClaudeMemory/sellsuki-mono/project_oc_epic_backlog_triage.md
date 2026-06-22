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

**Done 2026-06-22:** closed 26 verified safe-tier dead-shell epics → Done: OC-1739, OC-4045, OC-3866, OC-1571, OC-33, OC-32, OC-36, OC-22, OC-31, OC-101, OC-35, OC-1454, OC-209, OC-1565, OC-18, OC-1981, OC-1999, OC-2005, OC-2001, OC-1966, OC-1729, OC-1750, OC-2237, OC-2006, OC-2185, OC-2057. (Transition to Done = id `31`, global. OC statuses: To Do/In Progress/Blocked/Done only — no separate Closed.)

**Still un-triaged (~58 epics the po agents never analyzed):** block OC-1740–OC-1761, plus OC-5/9/26/27/34/38/39/41/44/57/59/222/223/1415/2224/2241/4130/4223/4224. Triage these before further bulk-close.

**⚠️ MCP Jira bug (recurring):** the Atlassian MCP server returns cross-project contaminated rows under load (PAT-xxxx issues appear in OC queries) and truncates/mis-paginates. `searchJiraIssuesUsingJql` ignores the `fields` restriction and returns huge payloads → use jq on the saved tool-result file. Don't trust agent-aggregated Jira counts; verify key lists with direct `key in (...)` JQL. The kill-list in an earlier round wrongly claimed OC-4200/4225/4283 don't exist — they DO (they're real epics on page 2). See [[reference_oc2plus_jira_project]].
