---
name: feedback-nested-po-lead-loses-drafts-write-to-files
description: Nested orchestration (po-lead spawning writers) gets force-reported mid-work and loses sub-agent output; drive writers from the main session and make them write drafts to repo files
metadata:
  node_type: memory
  type: feedback
  originSessionId: 9043b5b7-a828-4cca-b908-2d5347a98fbf
  modified: 2026-10-07T01:41:59.538Z
---

On 2026-10-06, running `/po-team` (po-lead → po-researcher/po/po-reviewer) to cut 16+ stories failed three times the
same way: po-lead was forced to hand back a status report while its children were still running, its children's full
markdown drafts stayed in po-lead's context and were lost, and the agent finally died on an API error. Root cause is
structural, not a prompt problem: a sub-agent's hand-back is the only durable channel, and nested agents report to
their parent, not to the session.

**Why:** chat is not storage (shipping.md §5) applies to agent transcripts too; a report that scrolls away or a child
whose parent was cut off is work that never happened.

**How to apply:** for multi-card drafting, spawn writer agents **directly from the main session**, one per epic/file,
and have each **write its draft to a repo file** (`docs/cards/drafts/<area>/<EPIC>.md`, cards delimited by
`=== CARD <epic> #<n>: <title> ===`) and commit; reviewers read the files; a single publisher agent with Bash `sleep 10`
creates Jira issues from the files (create with a short description, then `editJiraIssue` with `contentFormat:
markdown`, count ACs after). Use po-lead only for the final publish/link step or not at all. Reviewer-found cross-card
gaps go into a `REVIEW-round<n>.md` next to the drafts so fixers read one file. See [[project-sukispace-marketplace-ais-vision]].
