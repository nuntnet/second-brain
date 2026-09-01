---
name: reference_ccs_global_config_permission_gate
description: "central-configuration-system requires sellsuki.configsystem.config.view on tenant sellsuki.user:\"\" even for global-scope (empty userId/location) config reads"
metadata:
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-09-01T04:57:09.356Z
---

`central-configuration-system`'s `GetConfiguration` use case (`src/use_case/configuration.go:59-66`) checks permission `access_control.PermissionSellsukiConfigsystemConfigView` scoped to `entity.SellsukiUser` at the **`userId` query-param value** — even when that config is documented as global-scope (e.g. `portal-app-registry`, called with `?userId=&location=`). This means the tenant checked is the degenerate `sellsuki.user:""` (empty id), and **every** caller — not just per-user configs — needs an explicit RPS/Keto grant of `sellsuki.configsystem.config.view` on that exact empty-string tenant, or every `GET /v1/configuration/{service}` call 403s regardless of the config's own scoping semantics.

**How to apply:** if a frontend's first-ever CCS integration returns 403 `permission_denied` locally/in a fresh env, check this first — it's very likely nobody has granted `sellsuki.configsystem.config.view` on `sellsuki.user:""` to the acting identity yet, not a bug in the new integration code. Grant via RPS's `CreateRole`+`AssignRole` (gRPC, `role_and_permission.RoleAndPermissionService`) with `tenant: {kind: "sellsuki.user", id: ""}`. See [[reference_local_kratos_identity_debugging]] for how to find the *correct* identity to grant it to — guessing from `/admin/identities` is unreliable.

RPS also caches `CheckPermission` results in Redis (5 min TTL) — after granting a role, if a request still 403s, call `ClearPermissionCache` (RPC, takes `tenants: [{kind, id}]`) rather than waiting out the TTL.

**Confirmed this gate is per-environment, not just local:** the exact same 403 hit staging (`api.staging-th.sellsuki.com`) for a first-time CCS integration there too — grant separately per environment (local/dev-th/staging/prod each have their own RPS+Keto state).

**`GET /v1/schema/{service}` is gated by the same permission check as `GetConfiguration`** — so you can't use "does the schema even exist" as a permission-independent diagnostic. Confirmed by 403 `permission_denied` on `/v1/schema/portal-app-registry` with the exact same identity that 403s on the config read. Get the permission granted first, *then* re-check schema/config existence in one shot — a 404 `schema_not_found` after that means the service's `0002_seed_*`/`0003_seed_*` migrations haven't been run against that environment's etcd yet (see `backend/central-configuration-system/cmd/migration/migrations/`).
