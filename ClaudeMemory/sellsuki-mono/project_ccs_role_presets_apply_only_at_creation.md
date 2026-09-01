---
name: project-ccs-role-presets-apply-only-at-creation
description: "CCS applies model.CompanyOwnerPermissions only when a company is created — adding a permission to the preset reaches new companies and nobody else, so every such change needs a backfill run"
metadata:
  node_type: memory
  type: project
---

`sellsuki-central-control-backend` applies its role presets
(`src/use_case/model/permission_group.go` → `CompanyOwnerPermissions`, role name
`model.RoleOwner` = **"Company Owner"**) **only at company creation**
(`src/use_case/company.go`, `generic_logic.go`). Adding a permission to the
preset therefore reaches newly created companies and no existing one — every
company that already exists keeps the role it was created with.

The role API cannot fix it either: those roles are `IsSystem: true` and
`role_update.go` returns `ErrSystemRoleNotAllowed` — see
[[reference-rps-is-system-role-trap]].

**Why:** confirmed 2026-09-01 — `oc2plus.apikey.manage` was in the staging
permission catalog and the CCS3 toggle rendered, but the API-key page still
returned **403** because the existing company's Company Owner role predated the
preset change.

**How to apply:** every preset change ships with a run of
`cmd/backfill_role_permission` in the rps repo (dry-run by default, `--apply`
writes, idempotent). It goes through the repository layer so both the Postgres
array and the per-tenant-ref Keto tuples are written; a role with no tenant refs
is reported, not written. Needs `DATABASE_ROLE_POSTGRESQL_URI` +
`ORY_KETO_{READ,WRITE}_GRPC_SERVER`, so a port-forward is enough — it does not
need to be deployed. Landed as rps !111 (develop) / !112 (main); rps has two
live mainlines, see [[reference-rps-dual-mainline]].
