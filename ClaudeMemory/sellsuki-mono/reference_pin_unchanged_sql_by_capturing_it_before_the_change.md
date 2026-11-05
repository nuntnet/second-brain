---
name: reference-pin-unchanged-sql-by-capturing-it-before-the-change
description: "To prove \"no filter = today's query\" in chat-core, capture the repo method's SQL with testdb.SQLCapture on the untouched code first, then assert exact equality after the change"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0a457a4-bdb2-49cb-9feb-4d55ceac216f
  modified: 2026-11-05T10:43:57.496Z
---

When a change adds an optional filter (AI-397: `WorkspaceCounts(caseType)`) and the AC says the unfiltered read must stay exactly as it was, write a throwaway test **before editing**. In it, wrap the DB in `testdb.SQLCapture` (`c.Session(db)`), call the method, and print the whitespace-normalised `c.Matching("<table>")`. Paste those strings into the real PIN test as constants, with a comment naming the commit they were captured from. Then make "no filter" a `nil` scope func, so it adds no `.Where`. The pin then catches a predicate that slips in, and fails with a named FAIL when proving red.

Also: the Claude Code safety check refuses `rm -f $VAR/*`. Use `"${VAR:?}"/*`, or a new literal directory.

Related: [[reference-chat-core-integration-tests-share-one-migrated-db]]
