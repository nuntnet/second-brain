---
name: chatcore-ragcore-two-hops
description: "chat-core talks to rag-core over TWO independent hops with different credentials — the console KB hop (HS256 BFF token) works, the reply-path retrieval hop (static bearer) cannot authenticate at all. Treating them as one is why the KB looked fully blocked."
metadata:
  type: reference
---

Verified 2026-09-20 by reading both sides.

| composition-root fn | env vars (all three required together) | state |
|---|---|---|
| `initKbEntryGateway` — admin console KB CRUD | `RAG_CORE_BASE_URL` + `RAG_CORE_PROVIDER_ID` + `RAG_CORE_BFF_JWT_SECRET` | **workable** |
| `initRAGSource` — reply hot path retrieval | `RAG_CORE_BASE_URL` + `RAG_CORE_PROVIDER_ID` + `RAG_CORE_AUTH_TOKEN` | **blocked** |

Both in `cmd/chat_core_server/helper.go`, and it checks each trio
**independently** — so setting the first three and leaving `AUTH_TOKEN` unset
lights up the console half only. That is not the "half-set trio" the values
file's own comment warns about.

## Why one works and the other does not

- **KB hop**: `src/repository/rag_core_client/auth.go` mints a 60s **HS256**
  token per call over a secret shared with rag-core, with its own `iss`/`aud`.
  rag-core accepts it via `BFFJWTIdentityResolver`
  (`src/repository/identity_repository/bff_jwt.py`), chained in front of the
  gateway resolver when `BFF_JWT_SECRET` is set. No channel-gateway key needed.
- **Retrieval hop**: `src/repository/rag_source/http.go` sends a **static**
  bearer. rag-core runs `AUTH_MODE=gateway_jwt`, so that must be an RS256 JWT
  signed by channel-gateway — which chat-core has no key for and cannot mint.
  `rag_source/doc.go` records this as "gap 1" and it is still open.

Vault key names differ across the hop for the same value:
rag-core reads `BFF_JWT_SECRET`, chat-core reads `RAG_CORE_BFF_JWT_SECRET`.
One HMAC secret, two names — they must match or every KB call 401s.

## 🔴 The trap in "just mint a BFF token for retrieval too"

rag-core's `workspace_scope.py::resolve_reader` sets
`may_read_internal = actor.actor_type in {"employee"}`, and there is **no
`service` actor type**. A service authenticating as `employee` can therefore
pull `internal` (staff-only) chunks into a **customer-facing** reply — which
rag-core's own comment forbids in as many words ("A service is deliberately
absent... a customer-facing answer must never be built on staff-only material").

So closing gap 1 needs a `service` actor type outside `_INTERNAL_READER_TYPES`,
not just a token. That is a cross-service authorization decision — see
`.claude/rules/design-doc-authority.md` §3.

Related: [[reference_kb_entries_three_blockers]] ·
[[reference_boot_guard_pins_an_old_image_silently]] · [[reference_rag_core_visibility_tiers]]
