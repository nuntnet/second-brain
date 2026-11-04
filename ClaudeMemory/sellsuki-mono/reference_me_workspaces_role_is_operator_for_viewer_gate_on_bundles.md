---
name: reference_me_workspaces_role_is_operator_for_viewer_gate_on_bundles
description: "Console - /me/workspaces returns role \"operator\" for an AI Chat Viewer too; any gate on the role string (or membership) lets a viewer see actions the server 403s. Gate on AI-360 bundles via useMyPermissions"
metadata:
  node_type: memory
  type: reference
  originSessionId: f746364c-7c1d-4189-93bc-f067e13be78d
  modified: 2026-11-04T03:22:49.497Z
---

`GET /me/workspaces` (the console's `AccessibleWorkspace.role`) only knows `super_admin | company_admin | operator`. An rps "AI Chat Viewer" (workspace.read only) comes back as `operator`. So `role === "operator"`, "any membership", or the 3-value `WorkspaceRole` type all treat a viewer as an operator.

Found 2026-11-04 by the integrated console verifier (real mode): a viewer saw "Take over" in the inbox and chat-core answered 403. Fixed on the redesign wave (console `6115a20`): the inbox and unsorted pages read `useMyPermissions(ws).has("operate" | "supervise")` (`GET /v1/workspaces/:ws/me/permissions`, AI-360, fails closed). Tests use `apps/admin/src/test/permissionsPort.ts` to say what the user holds.

**How to apply:** never gate a chat action on `role` or membership; use the bundle (`useMyPermissions`). `canManageWorkspace` (role string, 7 pages) is the same bug, still open as of 11-04. When resolving a merge between two gates, check what the API actually returns for each role, not what the type allows — "the types are the same set" was true of the type and false of the data.
