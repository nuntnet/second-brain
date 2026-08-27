---
name: rps-identity-kind-must-be-prefixed
description: "rps rejects any Identity.Kind or tenant kind outside entity's allowlist — bare \"user\"/\"chat_workspace\" answer InvalidArgument \"identity invalid\", and nothing in a Go build can see it"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 270e02aa-6ae1-461a-ada0-6fb19a8e9e87
  modified: 2026-08-27T13:24:06.550Z
---

`sellsuki-role-and-permission-management-backend` validates **every** kind it
receives — both `Identity.Kind` and the tenant `Namespace.Kind` — against
`backend/entity`'s allowlist, and answers a single opaque
`InvalidArgument: identity invalid` for anything outside it. Verified directly
against a running rps on 2026-08-27:

| sent | result |
|---|---|
| `sellsuki.user` | ✅ accepted |
| `user` / `users` / `USER` | ❌ identity invalid |
| `sellsuki.chat_workspace` | ✅ accepted |
| `chat_workspace` | ❌ identity invalid |

**Why this is a trap:** a bare string compiles, type-checks and unit-tests
perfectly — the failure exists only at runtime, against a real rps. chat-core
shipped `Kind: "user"` at five call sites and `TenantKindChatWorkspace =
"chat_workspace"`; neither was caught by anything.

**How it presents:** the rejection also hits the READ path
(`ListAssignedTenants` → `ListAssignedRoles`), so `/v1/me/workspaces` answers
**503 `workspace_lookup_failed`** on every request rather than an empty list —
which reads as a deployment outage, not a wiring bug. On the WRITE path
(`AssignOperator`) the grant is simply never created, and the only symptom is
"the operator I just added still cannot open the workspace".

`entity.SellsukiUser` ("sellsuki.user") exists as far back as **entity v0.22.0**,
so use the constant — no version bump needed. The chat-workspace tier
(`SellsukiChatWorkspace`, AI-182) only landed in v0.31.0, so on a service still
pinning v0.22.0 spell `"sellsuki.chat_workspace"` literally with a note.

**Check both sides agree.** chat-core had the write path on `chat_workspace`
and the read path on `sellsuki.chat_workspace` — each package internally
consistent, each green, and a grant written where nothing looked for it. Bind
them with a guard in `src/guard` that reads both constants out of the source
(`go/parser`), not by importing both packages: `use_case` must not import a
concrete repository, and this repo's boundary guards **skip `_test.go` files**,
so an import added from a test slips through unnoticed.

Related: [[entity-lib-tenant-kinds]], [[chatcore-unprefixed-chat-workspace]],
[[flag-without-enforcement]].
