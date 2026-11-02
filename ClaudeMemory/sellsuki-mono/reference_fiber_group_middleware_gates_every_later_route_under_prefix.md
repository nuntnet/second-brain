---
name: reference_fiber_group_middleware_gates_every_later_route_under_prefix
description: "Fiber app.Group(prefix, gate) mounts the gate as a Use on the WHOLE prefix — a second group on a shared prefix (e.g. an admin/supervise group) also gates every route registered after it there; and a helper that writes a 403 returns nil, so its caller must get a handled flag"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2ed39e79-b1a0-4aac-9faa-c61a3445c73a
  modified: 2026-11-02T11:46:30.505Z
---

Found 2026-11-02 (AI-360, chat-core): `admin := app.Group("/v1/workspaces/:workspace_id/conversations", gate(supervise))`
registered for ONE assign route also gated `GET /conversations/:session_id`, the websocket, follow-up intents and
SLA snooze (registered later under the same prefix) — operators got 403 there, pre-existing since the company_admin
days. Only a route × actor matrix test over the real composition caught it.

**How to apply:** gate per route (`group.Post(path, gate, handler)`) whenever a prefix is shared; never add a
second `app.Group(sharedPrefix, gate)`. A test that enumerates `app.GetRoutes(true)` and asserts each route's gate
per actor (chat-core `src/interface/fiber_server/permission_matrix_test.go`) is the guard.

Second trap from the same card: a helper that does `return c.Status(403).JSON(...)` returns a **nil error**, so
`if err := helper(); err != nil { return err }` carries on and runs the write under a 403 status (the kill switch
was re-enabled by a team lead this way). Return `(handled bool, err error)`.
Related: [[reference_fiber_ctx_strings_corrupt_prometheus_labels]].
