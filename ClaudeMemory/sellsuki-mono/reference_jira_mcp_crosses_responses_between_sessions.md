---
name: reference_jira_mcp_crosses_responses_between_sessions
description: "The Atlassian MCP can return ANOTHER concurrent Claude session's tool result to your call — same sessionId, wrong toolName, wrong issue. Verify by querying, never by reading the response you got back"
metadata:
  node_type: memory
  type: reference
---

Observed 2026-09-04 with two Claude sessions working the same workspace.

I called `createJiraIssue` for project **PAT**. What came back was a full issue
payload for **AI-179** (project AI), with `"toolName":"transitionJiraIssue"` in
the response `context` block — a tool I did not call, on an issue I was not
touching. The other session was transitioning AI-179 → Done at that same second.

Both responses carried the **same** `sessionId` (`r11-…`). The Atlassian MCP
appears to key in-flight requests per authenticated user, not per client
session, so two concurrent sessions on one Jira account can have their replies
crossed.

## Why this is dangerous rather than merely confusing

Reading that response literally leads to two opposite wrong conclusions:

- *"my create failed"* → retry → **duplicate card**
- *"I transitioned AI-179 to Done"* → post a correction, or revert someone
  else's legitimate transition

Both sessions authenticate as the same Jira user, so the **changelog cannot
tell you which session did what**. Author and timestamp are identical in kind.

## What to do

**Never treat a Jira MCP write's response as evidence the write happened.**
Confirm with a separate query keyed on something you control:

```
searchJiraIssuesUsingJql: project = PAT AND summary ~ "<a distinctive phrase>"
```

In this case that showed exactly one **PAT-2689** created at 21:37:56 — the
create had in fact succeeded, and only the reply was crossed. No duplicate, no
accidental transition.

Also: `created >= -1h` is **not** valid JQL here — it fails with "Expecting
operator but got '&'" because the tool HTML-escapes `>=`. Match on summary text
instead.

Compounding factor already known: this MCP dislikes parallel calls
([[jira-mcp-search-quirks]]). Serial calls do not prevent this one — the other
*session* is the concurrency, not your own tool use.

Related: [[parallel-sessions-git-safety]] (same two-session hazard, git side),
[[pat2658-reference-collision]] (the task this happened during).
