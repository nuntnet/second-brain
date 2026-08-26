---
name: reference-rps-list-assigned-roles-reverse-lookup
description: "rps CAN answer 'which tenants does this user hold a role in' via ListAssignedRoles — the widely-copied GAP note claiming no such endpoint exists is wrong"
metadata:
  node_type: memory
  type: reference
---

`sellsuki-role-and-permission-management-backend` exposes:

```protobuf
rpc ListAssignedRoles(ListAssignedRolesRequest) returns (ListAssignedRolesResponse)
  // request:  Identity user
  // response: map<string, RoleIdList> assignments   // "kind:id" -> role ids
```

That is a **reverse tenant lookup** — exactly "which tenants does this user hold a role in".

**Why this matters:** `ai-chat-admin-frontend/packages/core/src/permissions/permissionsPort.ts` has carried a GAP note since AI-92 saying *"neither AI-12 nor AI-17 has shipped a 'list workspaces the current session can access' endpoint"*, and it was read by everyone — me included, for most of 2026-08-25 — as **cannot be built**. It could have been built at any point. Built it as chat-core `GET /v1/me/workspaces` (AI-137, MR !25).

**How to apply:**
- Keys are `"kind:id"` — split on the FIRST colon only; kinds never contain one, ids may.
- Derive the coarse role from the **tenant tier**, not from a role id: rps has no lookup-role-by-name/id API, and the tier already says it (`sellsuki.company` = whole company, `sellsuki.chat_workspace` = one workspace). An id→name map would be a second source of truth.
- `sellsuki.company` has ALWAYS been in entity's `IsActor` allowlist, so company-tier answers work today with no dependency on [[project_ai_backlog_gap_sweep_202608]]'s tenant-tier work.
- Before believing any GAP note in this monorepo, check whether the RPC/endpoint it says is missing actually is. Several are not.
