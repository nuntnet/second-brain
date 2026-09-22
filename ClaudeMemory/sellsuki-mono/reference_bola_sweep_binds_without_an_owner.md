---
name: reference_bola_sweep_binds_without_an_owner
description: "OC2Plus's periodic BOLA bind sweep passes ownerUserID=\"\" by design, so grantBolaWorkspaceOwner never runs — 5,680 of 6,576 active staging bindings point at a BOLA workspace with ZERO admins that no human can ever open"
metadata:
  node_type: memory
  type: reference
---

Found 2026-09-22 chasing "BOLA บอกว่าไม่มีสิทธิ์เข้า workspace" on staging.

`backend/oc2plus-line-crm-service-backoffice-api/src/use_case/bola_binding.go`:

```go
// :144
if ownerUserID != "" {
    u.grantBolaWorkspaceOwner(ctx, sp, companyID, workspaceID, ownerUserID)
}
// :444 — the sweep for companies with no binding at all
u.BindBolaWorkspace(ctx, companyID, "")
```

The sweep has no requesting user to attribute, so it passes `""` **deliberately**
(the doc comment at :41 says so). The workspace is created and the binding is
marked `active` — and the admin grant is skipped **silently**: no WARN, no span
event, unlike every other failure path inside `grantBolaWorkspaceOwner`, which
logs. So the only trace that the owner was never granted is the empty
`owner_user_id` column.

Result, staging 2026-09-22:

| | |
|---|---|
| active bindings | 6,576 |
| …with empty `owner_user_id` | **5,680** |
| BOLA workspaces with 0 rows in `admins` | 5,126 / 7,514 |

A workspace with zero admins cannot be opened by anyone, ever — BOLA web answers
"You do not have access to that workspace", correctly. This is
[[project_bola_binding_never_worked_via_ccs]]'s cousin and is OC-4200's owner
lockout returning through the sweep OC-4544 added; OC-4385 fixed only the path
that has a real user.

Carded as **OC-4591**. The admin-facing half (no screen shows which workspace a
company is bound to, and no way out when you lack access) is **OC-4592**.

Two more facts worth keeping:

- `workspaces.company_id` in BOLA is **almost never written** — 3 rows out of
  7,514 on staging. Do not treat it as the company↔workspace link; the link
  lives in `staging_oc2plus_crm.oc2plus_bola_bindings`. The direct
  `bola_workspace_repository.Rest.Create` takes `companyID` as a parameter and
  never sends it (`createBolaWorkspaceRequest` is Name/Slug/PlanID only).
- The `company_id` in an OC2Plus member-api URL such as
  `/v1/company/d9gelm1kpjgm5ha5t7m0/members/otp` is the binding **slug** (xid),
  not the CCS company uuid. Join on `oc2plus_bola_bindings.slug`.
