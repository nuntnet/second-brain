---
name: reference-rps-is-system-role-trap
description: "In role-permission-service, is_system_role=true is NOT a small set — every company's seeded roles carry it, so filtering on it alone scans thousands of rows and ListRoles fans out per-role-per-permission"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T12:58:59.145Z
---

`is_system_role = true` in rps does **not** mean "the handful of global platform presets". Every company created through CCS seeds its own role set (Company Owner / Manager / Marketer / Warehouse / Store Admin / Finance Manager) with `IsSystem: true` — so on staging the flag matches **thousands of rows**.

Compounding it: `role_management_use_case.ListRoles` calls `populateGroupScopes`, which looks up **each permission of each returned role** individually. So a query that returns N roles issues O(N × permissions) lookups.

**Why:** hit 2026-08-11 — `resolveSystemRoleIDByName` (rps `/internal/v1/roles?name=…`, code I wrote) listed all `is_system_role=true` roles and matched the name in memory, with a comment asserting the set was "single digits". Real behavior: the endpoint hung for minutes, every BOLA staging invite timed out and fail-closed with 503. Fixed by adding `RoleFilterOptions.Name` pushed into SQL (MR !97/!98).

**How to apply:**
- Never use `IsSystemRole` as the only narrowing filter on a hot path. Filter by `Name` and/or `Owner` too — `uk_roles_name_owner (name, owner_id, owner_kind)` has `name` leading, so `WHERE name = ?` seeks.
- When a call through this service "hangs" rather than errors, suspect an unbounded `ListRoles`, not the network. Bisect order that worked: liveness through the Service (proves DNS + routing + pod) → the specific endpoint by pod IP with a long timeout (proves it's the handler).
- A downstream 503 from bola-backend's invite path means fail-closed on rps, not that rps is down — see [[project-bola-kratos-sso-staging]] and [[reference-rps-dual-mainline]].
