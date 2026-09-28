---
name: dev-rps-membertier-missing-0025-present
description: "On dev rps (2026-09-28), oc2plus.membertier.* is absent while 0025 invitation.manage is present — someone applied it off the record. /member-tier is read-only for everyone. Auto mode blocks Claude from writing rps grants; the user runs the script."
metadata:
  node_type: memory
  type: reference
  originSessionId: 78814507-bb88-4d16-a0fe-3ce06ba66f45
  modified: 2026-09-28T06:27:31.794Z
---

Checked read-only on 2026-09-28 13:10 with rps gRPC `GetPermissionByCode` on dev (ns sellsuki-dev):
- `oc2plus.membertier.manage` / `.override` → `Internal: not found`
- `oc2plus.member.invitation.manage` (0025, OC-4583) → present. It was NOT there on 2026-09-26 15:55, and nothing in `docs/release/environments.yaml` records who added it.

The two migrations have separate Ids (0025's folder is `0025_…` but its Id is `0024_add_oc2plus_member_invitation_manage_permission`). A normal headless run would have applied both, so someone either ran only one of them or created the permission by hand.

Consequence: https://crm.dev.oc2.plus/member-tier opens read-only for every company. The page gates edits on `membertier.manage`, and the backend is fail-closed.

Company `fe621d66-0255-4647-b1de-6ae081b6864f` is the user's dev test company. Its Company Owner role is `89586` and is a system role, so a grant must go through `MigrateRoles`, not `UpdateRole`.

**Auto mode blocks Claude** from calling rps `CreatePermission` / `MigrateRoles` (reason: Permission Grant). It blocked even editing the grant script afterwards. Write the script for the user to run, then stop; don't retry the call through another route.

Script: `scripts/rps-grant-membertier-dev.sh`. The user ran it 2026-09-28: both codes created, Company Owner 89586 of fe621d66 went 71 → 73, and Keto has tuples. Every other dev company and the CCS preset still lack it. Recorded in environments.yaml (commit 218aff6). Related: [[reference_rps_grant_permission_to_role]], [[project_ccs_role_presets_apply_only_at_creation]].
