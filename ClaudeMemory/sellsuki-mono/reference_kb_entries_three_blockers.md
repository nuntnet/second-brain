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

## Status 2026-09-03 — 2 of the 3 are closed, the last hop is not

Re-verified hop by hop, and the branch topology moved since the note above:

- **this app** `createChatCoreKbEntryAdapter` merged (AI-96) — blocker 2 gone.
- **chat-core** `route/kb_entry` + `repository/rag_core_client` (which calls
  `GET/POST/PUT/DELETE {base}/v1/rag/knowledge/entries`) merged to its `main`
  — blocker 3 gone, the BFF seam exists.
- **rag-core** still NOT merged, and it is not one branch: `GET /entries` (the
  admin listing) is on `feat/AI-43-service-actor` and
  `feature/AI-46-retrieval-acl`, while `feature/AI-43-kb-editor` carries
  POST/PUT/DELETE only. rag-core `main` has none of the four.

So the KB tab is wired end to end and returns **5xx against a rag-core
deployed from `main`**. Only rag-core merging AI-43 closes it.

⚠️ rag-core is **Python**, not Go. A `git grep --include='*.go'` sweep for
these routes returns empty, which reads exactly like "the endpoint does not
exist". It was the first thing that misled me here.

Two comments in the FE described this wrongly in opposite directions — main's
said no listing endpoint exists at all, MR !24's said the blocker was the
missing seam. Both closed; !30 states it per hop instead.

Related: [[rag-core-visibility-tiers]], [[chatcore-is-the-admin-bff]]
