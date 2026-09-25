---
name: reference_preset_copy_vs_shared_role
description: "Why every new permission needs a per-env migration: CCS copies a preset BY VALUE into one role row per company; rps/Keto would propagate fine — chat's single shared Operator role already proves it"
metadata:
  type: reference
---

Verified from code 2026-09-25. The "add a permission → migrate every old company
in every env" loop is **not a Keto limitation and not an rps one**.

**rps already propagates role edits.** `ketoPostgresqlRoleRepository.UpdateRole`
calls `updateKetoTuplesForRole`, which loops every `TenantRefs` entry of that role,
deletes its permission tuples and rewrites them from the new list
(`src/repository/role_repository/keto_posgesql.go` ~L883). Edit one role row through
rps → every tenant it is assigned at sees the change.

**CCS defeats that by copying.** Company creation / owner bootstrap does
`SetRole(Name: RoleOwner, Permissions: model.CompanyOwnerPermissions[:], Owner:
<company>, IsSystem: true)` (`use_case/company.go` ~L252, `generic_logic.go` ~L96).
Each company gets its OWN role row holding a snapshot of a Go array. N companies =
N frozen copies; changing the constant only reaches companies created afterwards.
Staging company f2bf928c's owner role is `sellsuki.role:605106`, used by that
company alone.

**Proof the shared shape works: chat.** `sellsuki-chat-core`
`role_assignment_repository/grpc.go` `resolveOperatorRoleID` resolves ONE
"Chat Workspace Operator" system role by name and assigns it at
`sellsuki.chat_workspace:<id>` for every workspace. Add a permission to that one
role → all workspaces get it, no migration.

Provider has a manual band-aid: `POST /admin/provider/{code}/refresh-role` re-SetRoles
the provider preset for ONE provider. Company has no equivalent.

~~rps caches denied results in Redis~~ — **WRONG, corrected 2026-09-25.** See
[[reference_rps_permission_cache_is_never_enabled]]: the cache code exists but the
service never passes it a Redis client, so nothing is cached in any environment.

Related: [[project_ccs_role_presets_apply_only_at_creation]] ·
[[project_pointclaim_permission_missing_from_owner_preset]] ·
[[reference_rps_is_system_role_trap]]
