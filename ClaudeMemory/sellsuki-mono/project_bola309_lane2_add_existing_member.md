---
name: project_bola309_lane2_add_existing_member
description: BOLA-309 lane 2 — decided 2026-09-09 to grant the rps role directly (option A) when the invited email already has a usable identity, instead of mailing an invitation
metadata:
  type: project
---

Decided **2026-09-09** (user chose option A): when the email being added to a BOLA workspace already belongs to someone who can sign in, grant the rps role directly and create the admin row **Active** — no invitation, no email. Someone with no identity yet still gets the invitation.

**"สมัครเสร็จก็ได้สิทธิ์" already worked** before this: `Authenticate` auto-activates a pending admin on first login, on both the identity-UUID and cookie paths (`src/repository/auth_provider/kratos.go`). Nothing was needed for that half.

**BOLA enforces from its own `role` column** — 110 `RequireRole` call sites in route files; Keto `CheckPermission` appears only in `permission.go` / `rbac.go`. So an Active row with the right role is fully usable inside BOLA; the rps/Keto tuple is what makes the member visible to CCS3. That is why option A (direct grant) rather than "BOLA-local only".

The grant **blocks** on failure, unlike `UpdateAdmin`'s best-effort `AssignRole`: there a failed sync only means BOLA and Keto disagree about the *level* of access, here the grant is the only thing conferring access at all.

Key correction recorded: `provision-admin` does **not** leave the row pending — it passes `ActivateImmediately: true` and `ProvisionAdmin` activates an existing pending row idempotently. An earlier note claiming an email round trip + pending row came from reading a stale submodule checkout (`feat/oc-4514-oc2plus-standard-menu`, 834 lines behind `origin/main`) — see [[reference_grep_stale_branch_not_origin]].

MRs: bola-backend **!170**, bola-frontend **!113** (both off `main`, unmerged as of 2026-09-09). Depends on [[reference_kratos_identity_exists_is_not_registered]] and [[reference_rps_internal_endpoints_confirmed]]. Follows [[project_bola309_invite_via_ccs]].
