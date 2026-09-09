---
name: project-bola309-invite-via-ccs
description: "BOLA-309 §6a second half (decided 2026-08-14): BOLA offers 'Invite member' itself and calls CCS with the acting admin's X-User-Id — no service token; CCS's company-only authz was widened to accept admin.manage on bola.workspace. BOLA side merged+deployed; CCS side still on an unmerged stack as of 2026-09-09"
metadata:
  node_type: memory
  type: project
---

Extends [[project_bola_saas_access_model]]. §6a split Team Members **per action**
(CCS3 owns creating an identity, BOLA owns role/activate/remove for someone already
in the workspace). This is the decision about the remaining half — inviting a new
person — chosen by the user on 2026-08-14 over "BOLA creates the Kratos identity
itself".

**Shape:**

```
BOLA /team-member → POST /v1/workspaces/:id/admins/invite-via-ccs
                  → CCS  POST /v1/bola-workspaces/by-bola-id/:id/invitations
                  → rps mints an invitation scoped to bola.workspace only
```

- **No service token.** BOLA forwards the acting admin's Kratos identity in
  `X-User-Id`; CCS authorizes *that* identity against Keto. So BOLA never vouches
  for its callers and nobody can invite through BOLA what they could not invite
  directly. `CCS_BASE_URL` unset = "no CCS here" (self-host), reported as
  unsupported rather than as an outage.
- **CCS authz had to widen**, and that was the actual blocker: `InviteToBolaWorkspaces`
  required `PermissionSellsukiCompanyUpdate` on the **company**, which refuses
  exactly the workspace super_admin the feature is for. Company permission stays
  the fast path; on denial it falls back to **`admin.manage` on
  `bola.workspace:<id>`** (seeded by rps migration
  `0015_seed_bola_workspace_role_presets`). The fallback runs **inside the resolve
  loop for EVERY selected workspace** — authorizing once then looping lets a caller
  bundle a workspace they do not administer.
- Keto stays the single authority; no new trust boundary.

**State as of 2026-09-09:**

- BOLA backend + frontend: **merged and on staging** (`bd02368` / `c9ccb0d`).
- CCS: **on neither `develop` nor `main`.** `!282` merged into
  `feat/bola-311-manage-workspace-invitations`; that stack (`!279`, `!281`, `!280`)
  is still open. So BOLA's live "Invite member" button calls an endpoint no deployed
  CCS serves.
- `CCS_BASE_URL` is in **no** `deployment/values-*.yml`, so the button would not
  even render on staging/prod yet.
- Open concern for SRE: CCS `/v1` trusts the `X-User-Id` header
  (`helper/auth.go`), i.e. it assumes an authenticating gateway in front. Adding
  BOLA as a caller makes that worth confirming (gateway or NetworkPolicy) before
  production. Pre-existing property of CCS, not introduced here.
