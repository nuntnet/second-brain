---
name: reference_bola_line_oa_channel_is_globally_unique
description: "A LINE OA channel_id is unique across ALL BOLA workspaces — an active registration elsewhere returns 409 naming that workspace, and only soft-deleting it there releases the channel; ProvisionAdmin cannot be used to get back into the holding workspace"
metadata:
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-29T11:44:50.893Z
---

**Read from the code 2026-09-29** (`backend/bola-backend`).

## The rule

`CreateLineOA` (`src/use_case/line_oa.go:62`) looks the `channel_id` up
**`Unscoped` and across every workspace**, then:

| state of the OA elsewhere | result |
|---|---|
| active (`deleted_at` null) | **409 `line_oa_channel_id_duplicate`** — refused, to prevent a cross-tenant read/write via upsert |
| soft-deleted | released automatically: the old row's `channel_id` is tombstoned to `<channel>#released#<oaID>` and a brand-new OA is created (no followers/history carried over) |

So the only supported way to move an OA between workspaces is to **delete it in
the holding workspace first**. No DB edit is needed for the move itself.

## The error already names the workspace

`ChannelConflictError` (`src/entity/line_oa/errors.go`) carries the holding
workspace's display name; `helper.SendError` surfaces it as a `workspace_name`
field, and `ConnectLineOADialog.tsx` renders it in Thai. Merged 2026-07-21, and
present in the staging image as of 2026-09-29 — **read the full error text before
going hunting in the database.**

## 🔴 ProvisionAdmin does NOT get you into the holding workspace

The obvious recovery — `POST /v1/system/workspaces/:id/provision-admin` — takes
an **email**, not a Kratos id, and `ProvisionAdmin` (`src/use_case/admin.go:361`)
only activates a row whose status is `pending`. An existing **active** row
returns `409 admin_email_taken` and changes nothing.

That is exactly the state left by the `local_jwt` → Kratos move: the admin row is
active with the right email, but `kratos_identity_id` is empty, and Kratos login
resolves people by `FindAdminsByKratosIdentityIDGlobal`
(`src/repository/auth_provider/kratos.go:204`), never by email. The row exists;
login cannot find it. The fix is to populate that column — see
[[reference_bola_backfill_kratos_ids_cannot_run_as_documented]].

## If you do patch a row by hand

Migration 0128 added a **partial unique index**
`uq_admins_workspace_kratos_identity_id` on `(workspace_id, kratos_identity_id)
WHERE kratos_identity_id <> ''`. Check no other row in that workspace already
holds the UUID, or the UPDATE fails. Also confirm `role = 'super_admin'` if the
person then needs to invite others. Tables are `line_oas` and `admins`.

Kratos admin API: ns `share` on staging, `share-dev` on dev, port 80 — never
`:4434` ([[reference_dev_th_cluster_access]]).
