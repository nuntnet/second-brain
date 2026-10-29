---
name: reference_local_second_identity_operator_manager_recipe
description: "Local ai-mvp: how to make throwaway Operator/Manager Kratos users, grant via chat-core POST /v1/workspaces/:ws/operators (works, role 738) or rps AssignRole, and clean up — rps UnassignRole leaves tenant_refs + permissions tuples behind"
metadata:
  node_type: memory
  type: reference
  originSessionId: 40b9b4d5-4db5-4b9e-88e8-be68804003e0
  modified: 2026-10-29T15:49:58.129Z
---

Done 2026-10-29 (sweep docs/cards/sweeps/2026-10-29-local-verify-s21.md "Second identity — local"):

- Identity: `POST 127.0.0.1:4434/admin/identities` schema `email`, traits {name{first,last},email,phone,ref_id,pdpa_reference_id}, `credentials.password.config.password`, verified address. Generate password in python, store only in gitignored `.claude/verify/.local/credentials.json`. Delete: `DELETE /admin/identities/<id>`.
- Login for curl: Kratos browser flow at :4433 with Accept: application/json, carry the csrf Set-Cookie by hand, take `sellsuki_session` from Set-Cookie (the API flow's session_token is NOT accepted by chat-core).
- Operator on one workspace: as the seed Company Owner `POST :8099/v1/workspaces/<ws>/operators {"user_id"}` → role 738 tuple `roles:sellsuki.role:738#sellsuki.chat_workspace:<ws>@sellsuki.user:<id>`. Revoke `DELETE …/operators/<id>`. Both work; the "write path rejected" comment in chat-core accessible_workspace.go is stale.
- Manager/other company roles: `grpcurl -plaintext localhost:9998 role_and_permission.RoleAndPermissionService/AssignRole` {role_id, tenant{kind,id}, user{kind:sellsuki.user}, actor}. Keto subjects are `sellsuki.user:<id>` (query `?subject_id=sellsuki.user:<id>` on :4466).
- Cleanup trap: rps `UnassignRole` removes only the member tuple. The first AssignRole of a role on a tenant also appended `roles.tenant_refs` and wrote one `permissions:<perm>#<tenant>@roles:sellsuki.role:<id>#<tenant>` tuple per permission; remove both by hand (Keto PATCH :4467/admin/relation-tuples + array_remove in db sellsuki_role_permission) after checking no other member holds that role×tenant.
- Single-workspace company for "last workspace" ACs: `/internal/workspaces/provision` + AssignRole role 17 on that company; delete `workspaces` + `workspace_audit_logs` rows after.

Related: [[reference_local_chat_core_api_drive_recipe_and_consent_gap]], [[reference_local_ai_chat_console_open_on_localhost_with_seed_identity]], [[project_test_users_as_infrastructure]]
