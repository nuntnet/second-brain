---
name: kb-entries-three-blockers
description: "AI-96's KB entries were blocked by 3 things, not the 1 the code marker named; rag-core's CRUD is on an unmerged branch"
metadata:
  type: reference
---

Verified 2026-08-29.

`createUnavailableKbEntryAdapter` said rag-core "has the endpoints already, the
browser just cannot authenticate". The auth half was **accurate and precise**
(Bearer gateway-JWT vs Kratos cookie; channel-gateway decommissioned;
`AUTH_MODE=fake` refused on staging/prod). It still understated the work by
two thirds:

1. **rag-core's `/entries` CRUD is NOT on rag-core main** — it lives on
   `feature/AI-46-retrieval-acl`, **28 commits ahead, not merged**
   (`src/interface/http_server/routes/knowledge_entries.py`). That branch mixes
   AI-43 (KB editor backend) and AI-46 (retrieval ACL), so they cannot ship
   separately.
2. **No chat-core route reached it** — `rag_source/http.go` proxies only
   `/v1/rag/knowledge/retrieve` (the reply path's read).
3. auth, as the marker said.

**And a test made it look safe**: it asserted the message contained
"authenticate" and "Kratos" — which it did — so the suite was green on a marker
that named one blocker of three. A test pins what a message SAYS, never whether
what it says is complete.

## What was built (chat-core BFF, AI-96)

`GET/POST/PUT/DELETE /v1/workspaces/:id/kb-entries` → rag-core. Company-admin
on reads too, mirroring rag-core's own note that entries can carry per-person
content.

**Two subjects get authorized and must not be conflated**: the browser user (by
chat-core/Keto) and the service token (by rag-core: `employee` +
`kb_editor_groups`). A rag-core 403 answers **503 `kb_not_entitled`**, never a
browser 403 — otherwise an admin goes asking for rights they already hold while
the fix is an env var. This is a pattern for every proxy here, not a one-off.

rag-core's `resolve_kb_editor_scope` only checks employee + a GLOBAL group; it
names AI-12 as the owner of real per-workspace entitlement. The BFF adds that
on the console path only — direct rag-core callers are unchanged.

Related: [[rag-core-visibility-tiers]], [[chatcore-is-the-admin-bff]]
