---
name: reference_chat_core_permission_matrix_needs_every_new_route
description: chat-core TestPermissionMatrix_EveryWorkspaceRouteIsInTheTable fails for any new /v1/workspaces route until it gets a route(row, method, path, bundle) line
metadata:
  type: reference
---
`src/interface/fiber_server/permission_matrix_test.go` (AI-360) lists every workspace route with its design row and bundle (`rd`, `op`, `pe`, `su`). A new route is red there until it has a line, and the test also checks the route's real gate matches the listed bundle. Add the row while you add the route; per-package `-run` tests never show it — only the full `./src/...` ladder does (found on AI-368 / AI-390, 2026-11-04). Sibling guard: each route package's `openapi_test.go` parity test. See [[feedback_wave_integration_full_suite_and_tsc_on_merged_tree]].
