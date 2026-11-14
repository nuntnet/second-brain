---
name: reference_jira_mcp_editissue_fields_rejected_as_string
description: "Atlassian MCP editJiraIssue loaded as a schema-less tool rejects every `fields` object (\"Expected object, received string\") — description cannot be edited; use a comment instead"
metadata:
  node_type: memory
  type: reference
  originSessionId: 17b79f90-4e6f-4d55-85bf-48f14625212a
  modified: 2026-11-14T06:43:45.464Z
---

2026-11-14 (AI-442): `editJiraIssue` arrived via ToolSearch with schema `{"type":"object"}` and no properties. Every call with `fields: {...}` (even just `labels`) failed: `Input validation error: fields: Expected object, received string`. createJiraIssue and addOrEditJiraIssueComment (string params only) worked.

**Update 2026-11-14 (later the same day):** a second Atlassian connector (`mcp__d4aa0ce4-…__editJiraIssue`, loaded via ToolSearch) has a full schema and accepted `fields: {"description": …}` with `contentFormat: markdown`; Thai and 19 AC bullets came back intact. Use that one for description edits.

**How to apply:** don't retry the schema-less edit; Post the change as a Jira comment (markdown renders fine), keep the repo ledger as the source of truth, and tell the user that the description is stale and needs a UI edit (or a session where the tool has its full schema). Related: [[reference_jira_editissue_adf_breakage]].
