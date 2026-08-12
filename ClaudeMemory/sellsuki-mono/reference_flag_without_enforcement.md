---
name: reference_flag_without_enforcement
description: "A state flag checked in one middleware but not its sibling enforces nothing — BOLA suspend was enforced on 11 routes and ignored on ~40; plus fiber c.JSON returns nil, so an error-only guard helper reads as 'allowed'"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-12T17:32:59.067Z
---

Two ways an enforcement flag ends up decorative. Both hit BOLA-310 on 2026-08-13.

## 1. Sibling guards drift

`bola-backend` has two auth middlewares. `WorkspaceGuard` (used where the route
has `:workspace_id`, **11** registrations) refused an inactive workspace.
`FlatAdminGuard` (**~40** registrations — broadcast, line_oa, follower,
ai_chatbot, media, rich_menu, lon, pnp: essentially the product) never checked.
So a suspended workspace kept working from any already-open session.

Nothing failed. Grep counts are the cheap detector:

```bash
grep -rn "FlatAdminGuard" src/interface/fiber_server/route/ | wc -l
grep -rn "WorkspaceGuard"  src/interface/fiber_server/route/ | wc -l
```

When two guards can authorize the same resource, the shared rule belongs in one
function both call — not copied into whichever one the author was editing.

**Consequence for tests:** moving the check into `FlatAdminGuard` means it now
loads the workspace on every flat route, so route tests that passed `nil` for the
workspace repository panic. That is correct (the guard genuinely needs it), but
expect to wire a mock into every route-test helper.

## 2. `c.JSON` returns nil

Fiber's `c.Status(403).JSON(...)` returns **nil** on success. A helper that writes
the refusal and signals it through `error` therefore looks like *allowed* to its
caller:

```go
if err := refuseIfInactive(c, uc, id); err != nil { return err }  // never fires
```

The request then falls through to `c.Next()`, the handler overwrites the body, and
the response carries a **403 status with a success payload** — which passes a
status-code assertion. Return `(ok bool, err error)` and branch on `ok`.

Related: [[reference_gorm_updates_drops_false]], [[reference_ambiguous_404_fail_open]]
