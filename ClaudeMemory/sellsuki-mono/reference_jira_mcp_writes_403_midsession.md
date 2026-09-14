---
name: reference_jira_mcp_writes_403_midsession
description: "Jira MCP writes can start failing 403 \"The app is not installed on this instance\" partway through a session while reads keep working — not a permission or payload problem, and retrying does not clear it."
metadata: 
  node_type: memory
  type: reference
  originSessionId: fa22ebb1-f715-4667-85a1-61ebbcc816ab
  modified: 2026-09-14T08:25:58.858Z
---

Jira MCP **writes** (`addCommentToJiraIssue`, `transitionJiraIssue`) can start
returning `403 {"code":403,"message":"The app is not installed on this instance"}`
**partway through a session**, while `getJiraIssue` and
`searchJiraIssuesUsingJql` on the same cloudId keep working normally.

Observed 2026-09-14 on OC-4549: `createJiraIssue` and `addCommentToJiraIssue`
both succeeded earlier in the same session, then every write started 403-ing
about an hour later. Retried four times across several minutes, with different
comment bodies (wiki markup, then plain text with no `{{}}` or table markup) —
same 403 every time. The `sessionId` in the response context changed between
calls, so it is not one poisoned connection.

**What it is not:** not an ADF/Thai-text problem (see
[[reference_jira_editissue_adf_breakage]]), not a field-permission problem, and
not a cloudId mistake — reads against that exact cloudId answer fine.

**What to do:** don't burn retries. Say plainly that the Jira write failed,
paste the comment text or the transition you intended into the chat so the user
can apply it, and keep going with the code work. Re-auth of the Atlassian
connector is the user's action, not something to attempt from here.

Related: [[reference_jira_mcp_crosses_responses_between_sessions]],
[[reference_jira_mcp_search_quirks]], [[reference_no_local_jira_fallback]].
