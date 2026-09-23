---
name: reference_rps_company_owner_lookup
description: "How to answer 'who owns company X' — rps GET /internal/v1/role-assignments by tenant plus the exact preset role name 'Company Owner'; and why checking a consumer's VENDORED proto makes rps look like it lacks the capability"
metadata:
  type: reference
---

**Verified 2026-09-23 while deciding OC-4591's open question** (who should own a
BOLA workspace when the sweep has no requesting user).

## The lookup

Tenant → its members and their roles, no user needed as input:

```
GET /internal/v1/role-assignments?tenant_kind=sellsuki.company&tenant_id=<uuid>
```

`route_internal.go` — the handler's own comment reads *"answers 'who holds which
roles inside this tenant'"*. Built for BOLA-315 reconcile. No caller auth
(network isolation) — see [[reference_rps_internal_endpoints_confirmed]].

The owner is the assignment whose role name is exactly **`Company Owner`**
(`cmd/backfill_role_permission/main.go` defaults `-role-name` to it). Role names
are unique per owner, so the name is a safe key.

Over gRPC the same answer comes from `ListUsersWithRoles`. **Its proto comment
lies**: `Identity scope` is documented as "User to list roles for", but the use
case passes it straight to `Aggregation.ListUsersInCompany(ctx, scope, …)` — it
is the TENANT. `ListUserRolesInTenant` (user + tenant → roles) is the other
direction, and `CountRoleMembers` returns only a count, no identities.

## 🔴 The trap: a vendored proto is not the service

I concluded in OC-4591 that this capability did not exist and that option 1 was
"cross-service work, needs a new RPC on rps". Wrong. I had checked
`oc2plus-line-crm-service-backoffice-api/src/repository/permission_repository/proto/`,
whose copy is **stale** — 20 RPCs, missing `ListUsersWithRoles` and
`ListUserRolesInTenant`. rps's own proto has them.

This is the same shape as `verify-at-the-boundary.md` rule 6 (filtering search
output is a claim about what you filtered): reading a *copy* and concluding the
*service* lacks something. [[reference_rps_list_assigned_roles_reverse_lookup]]
already ends with "before believing any GAP note, check whether the endpoint is
actually missing" — I had that note and still missed it, because the check I ran
was against the copy. **Check rps's own repo, always.** See
[[reference_rps_proto_is_vendored_per_consumer]].

## Cost for a consumer

`oc2plus-line-crm-service-backoffice-api` reaches rps over gRPC `:50051` only
(`ROLE_PERMISSION_GRPC_SERVER`, set in all three `values-*.yml`). To use the
internal HTTP route it needs a new base-URL env var added to those three files —
in-repo, no SRE ticket — and **no `envDefault` pointing at localhost**
([[reference_envdefault_localhost_masks_missing_config]]). The alternative is
regenerating the vendored stubs, which avoids the env var but is more churn.
