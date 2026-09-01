---
name: reference-rps-listroles-pointer-pagination
description: "rps ListRoles has TWO paging traps: LIMIT applies only to a *PageListOption (value = every row), and no sort option is unique so OFFSET paging over ties silently skips and duplicates rows"
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

## The worse one: no unique sort key

`ListRoles` sorts only by `name`, `created_at` or `updated_at` — **none unique**.
When the selected rows share a value (every `Company Owner` role does), the
`ORDER BY` gives no total order and Postgres may return tied rows in a different
order per statement. Paging with OFFSET across that **silently skips some rows
and repeats others**.

Measured 2026-09-01 on staging: two identical dry-runs three minutes apart over
the same 10,287 roles reported `already set` as **0** and then **343**. Same
data, different answer, no error either way.

**How to apply:** for a bulk read over rows sharing a sort value, do NOT page.
Read the count, then fetch everything in a single statement
(`&PageListOption{Limit: total + margin, Page: 1}`) and dedupe by ID — no OFFSET,
no shuffle. If you must page, pass `&PageListOption{...}` and
`&NameSortOption{}` (pointers) and bound by the returned `total`, but know the
result is not trustworthy for ties. See also
[[reference-rps-is-system-role-trap]] and
[[project-ccs-role-presets-apply-only-at-creation]].
