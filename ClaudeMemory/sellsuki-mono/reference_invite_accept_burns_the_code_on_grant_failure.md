---
name: reference_invite_accept_burns_the_code_on_grant_failure
description: rps commits the invitation usage in its own Postgres transaction before granting roles in Keto — a grant failure used to leave the invitee with no roles and a spent code; fixed by compensation, not reordering
metadata:
  type: reference
---

Accepting an invitation in
`sellsuki-role-and-permission-management-backend` is **two writes to two
systems**, and only the first is transactional:

1. `invitation_repository.AcceptInvitation` (postgresql.go) appends the actor to
   `usages_by` under `FOR UPDATE` and **commits**.
2. `role_management_use_case.AcceptInvitation` then grants roles in **Keto**.

A failure in step 2 therefore left the invitee with **no roles and a code already
marked used**. Every retry answered `/invite/status/expired`; only a new
invitation from an admin could fix it. Found by the CCS3 e2e run 2026-09-10 —
`TS-Accept-Invite-Link` looked flaky at 11/15 until the cases were split: **4 of
4 that actually press accept failed**, and none of the 11 passes ever pressed it.

**Fixed 2026-09-17 (MR !128 main / !129 develop) by compensation, not
reordering.** Moving the usage write after the grants is the obvious fix and is
wrong: the `FOR UPDATE` lock is what makes the quota check safe and it is
released at that commit, so two people could take the last seat. Instead
`ReleaseInvitationUsage` removes the one usage under the same lock.

Rules that came with it, worth keeping if this code is touched again:
- Grants that already succeeded are left in place — `AssignToRole` returns
  `ErrRoleAlreadyAssigned` and the loop continues, so a retry converges.
- Releasing an identity that is not in the usages is a **no-op, not an error**:
  the compensation runs on an already-failed path.
- If the release itself fails, log BOTH errors — the assignment error alone does
  not say the invitation was burned.
- The caller always gets the original assignment error, never the release error.

**CORRECTION 2026-09-17 — the staging 500 aborts EARLIER than this.** The dev
team's trace: `use_case.AcceptInvitation` (invite.go:310) calls
`assertInvitationAddressedTo` (invite.go:322 -> :260) FIRST, which does an HTTP
`GetInvitationDetail` and **fails closed**, so `AcceptInvitation` on rps
(invite.go:327) is never reached. The usage is therefore never written on that
path, and the compensation above does not fire for it — it guards a different,
still-real failure (rps reached, grants fail). Do not cite the compensation as
the fix for the staging 500.

**And the URL default was wrong everywhere, not just unset on staging:**
`envDefault:"http://localhost:8091"` — port 8091 is **address-backend**
(`Procfile: address: … PORT=8091`), while role-permission-service, which serves
`/internal/v1/invitations`, listens on **9999** (`role-perm: … PORT=9999
GRPC_PORT=9998`). So it never pointed at the invitation API in any environment,
local included. Fixed to :9999 plus a boot-time warning in CCS !334 (main) /
!335 (develop); the variable still has to be SET per deployed environment.

**Why it is REST and not gRPC** (asked every time): the gRPC contract cannot
serve the recipient check — neither the proto's `Invitation` nor
`model.Invitation` has an email or metadata field, and for BOLA workspace
invitations the address exists only in metadata. Moving it to gRPC means a proto
change in rps plus regeneration on both sides.

**Why the grant half was failing on staging at all:** CCS, not rps.
`sellsuki-central-control-backend/cmd/generics_server/main.go` declares
`INVITATION_API_BASE_URL` with `envDefault:"http://localhost:8091"` and the var is
set **nowhere in that repo** — so staging dials localhost and gets connection
refused. Same class as `BOLA_API_BASE_URL` / `MESSAGING_BACKEND_BASE_URL`, which
default to `localhost:8080` in the same struct. See
[[project_ccs_bola_provisioning_unwired]] — this repo has a standing habit of
service URLs that exist only as localhost defaults.

Related: [[reference_rps_dual_mainline]] (needs both MRs) ·
[[reference_lease_claim_ownership_bug_class]].
