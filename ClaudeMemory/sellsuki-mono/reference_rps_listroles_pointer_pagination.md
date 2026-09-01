---
name: reference-rps-listroles-pointer-pagination
description: "rps ListRoles applies LIMIT/OFFSET only for a *PageListOption — passing the value type silently returns every row on every page (infinite paging loop)"
metadata:
  node_type: memory
  type: reference
---

In `sellsuki-role-and-permission-management-backend`,
`src/repository/role_repository/keto_posgesql.go` → `ListRoles` applies
pagination through a type switch that matches **`*role_and_permission.PageListOption`**
(pointer). `PageListOption.ListOption()` has a **value receiver**, so a value
satisfies the interface and compiles fine — it just never matches the switch, so
no LIMIT/OFFSET is applied and every page returns the full table. The same trap
applies to the sort options (`*NameSortOption` etc.).

Symptom: a "page until an empty page" loop never terminates and grows memory
forever. Hit on 2026-09-01 writing `cmd/backfill_role_permission`.

**How to apply:** always pass `&role_and_permission.PageListOption{...}` and
`&role_and_permission.NameSortOption{}`, and bound the loop with the `total` the
call returns rather than trusting an empty page. See also
[[reference-rps-is-system-role-trap]].
