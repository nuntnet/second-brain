---
name: reference_rps_internal_endpoints_confirmed
description: All four rps /internal/v1 role endpoints exist (bola-backend's docs said otherwise), and the assign path does not validate tenant kind while the invitation path does
metadata:
  type: reference
---

Verified against rps `origin/main` on 2026-09-09, in `src/interface/fiber_server/route/v1/route_internal.go`:

- `GET  /internal/v1/roles?name=` → `{"role_id": N}`
- `POST /internal/v1/role-assignments` → assign
- `POST /internal/v1/role-assignments/unassign`
- `GET  /internal/v1/role-assignments?tenant_kind=&tenant_id=[&subject_kind=&subject_id=]`

All registered in `RegisterInternalRoutes`, **no caller auth** (network isolation only — hardening is a tracked follow-up that must land in lockstep across CCS3 + bola-backend). Field names are `role_id` / `role_name` / `tenant{kind,id}` / `subject{kind,id}`.

**bola-backend's `role_permission_repository` package doc used to call three of these "speculative — NOT CONFIRMED to exist in rps today".** That wording was true when written and outlived the gap; it was by itself a reason not to build on `AssignRole`. Corrected in MR !170.

Two behaviours that decide designs:

1. **Idempotent** for internal callers: `ErrRoleAlreadyAssigned` and `ErrRoleNotAssigned` both degrade to 200. Retry is safe.
2. **The assign path does NOT validate the tenant kind** — `AssignToRole` uses `tenant.ToString()` straight as the Keto relation. The **invitation** path runs `access_control.ValidateKind` and rejects unregistered kinds. Since `bola.workspace` is still unregistered in the shared entity package, a direct role grant works while a role-bearing BOLA *invitation* is blocked on the entity version bump. See [[reference_entity_lib_tenant_kinds]].

The subject kind IS validated (`identity.New` → IsActor registry), so it must be `sellsuki.user` etc. — see [[reference_rps_identity_kind_must_be_prefixed]].

Presets come from migration `0015_seed_bola_workspace_role_presets`; whether it is applied on staging is a runtime fact, not checkable from code.
