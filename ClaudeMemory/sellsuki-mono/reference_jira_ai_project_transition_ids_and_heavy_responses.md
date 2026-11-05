---
name: jira-ai-project-transition-ids-and-heavy-responses
description: "AI Jira project cloudId + global transition ids (In Progress 21, In Review 31); transitionJiraIssue returns the full description so bulk syncs go to subagents"
metadata:
  node_type: memory
  type: reference
  originSessionId: 5a53ee8c-9abb-4c2b-b876-f3fd655c972b
  modified: 2026-11-05T10:40:08.114Z
---

Jira site sellsuki.atlassian.net, cloudId `50dc7e4a-0539-4dcf-9c33-5481a395a5ab`. In the AI project the transitions are global: To Do `11`, In Progress `21`, In Review `31`, Done `41` (never use 41 on cards whose AC we wrote). The `transitionJiraIssue` result carries `fields.status.name`, so it doubles as the re-read.

**Why:** every comment and transition response echoes the whole card (about 9K tokens for the transition). A 45-card sync would burn the main context.
**How to apply:** for more than about 5 cards, hand batches of 7–9 cards to sonnet subagents, writes strictly sequential per agent, and have each check that the comment's `self` URL issue id matches the card. About 5 agents ran in parallel on 2026-11-05 with no 403/429 and no crossed responses. See [[no-local-jira-fallback]].
