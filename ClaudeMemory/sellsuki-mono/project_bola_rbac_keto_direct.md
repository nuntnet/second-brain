---
name: project-bola-rbac-keto-direct
description: "BOLA SaaS authz = Keto-direct (PERMISSION_PROVIDER=keto), NOT the CCS→Kafka→local sync; 3-tier model; sync MRs superseded"
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-07-21T17:06:53.220Z
---

**Decision (CTO, 2026-07):** BOLA authorization uses the pluggable `AdminRBACRepository` seam (`src/use_case/repository/admin_rbac_repository.go`, providers `inline`/`local`/`keto`, wrapped in `NewCached`). 
- **SaaS mode → `PERMISSION_PROVIDER=keto`** (query Sellsuki shared Keto at runtime, cached 30s) — this is **BOLA-118** (was a stub `keto_stub.go`, `NewKetoStub()` at `cmd/bola_server/helper.go:665`).
- **Self-host → `PERMISSION_PROVIDER=local`** (BOLA-205, DB-backed, full: custom roles + per-resource). Untouched.

**Rejected: the CCS→Kafka→local-table sync.** The sync consumer wrote CCS membership events into BOLA's **local admin table** (`adminRepository.CreateAdmin/UpdateAdmin` in `membership_sync.go`) → that's `PERMISSION_PROVIDER=local` fed by sync, which does NOT satisfy "SaaS uses Keto." Doing sync then Keto-direct = duplicate + two-sources-of-truth drift (same class as [[project_bola_contact_profile_model]] BOLA-173). So sync is dead:
- **MR !138** (bola-backend consumer) — MERGED but dormant (`CCS_MEMBERSHIP_SYNC_ENABLED=false`, auto_push_worker not deployed anywhere; migration 0128 applied harmlessly). Superseded → remove in a cleanup PR.
- **MR !251** (CCS3 producer) — CLOSED.

**3-tier authz model (BOLA owns `bola_workspace` Keto namespace + writes ALL tuples itself):**
- system admin (ops, all workspaces) = `BolaInstance:sellsuki#system_admin`
- company admin (company-bound workspaces) = `BolaCompany:{cid}#admin` via workspace→company link
- workspace super_admin (own workspace) = `BolaWorkspace:{ws}#super_admin`; + admin/editor/viewer
- Module perms (broadcast/media/etc) **derive from role via OPL subject-set rewrite** (NOT per-admin flat tuples — system admin can't tuple every module×workspace). Custom roles + per-resource = local only, unsupported in keto mode.

**One open decision (BOLA-118 Phase 1):** who writes `BolaCompany#admin` tuples — recommend **(a) BOLA writes them at company-bind/provision time** (OC-4385 provision path exists), not runtime dependency on CCS Keto.

**Why:** dual-mode provider already satisfies both requirements with one code path, zero sync infra. Keto has NO bola-workspace namespace today (role-service keeps BOLA roles as invite Metadata, `AllowNoRole=true`, F8 shipped) → BOLA must design+deploy the OPL namespace (infra deploys to shared Keto).

**How to apply:** any future "BOLA SaaS permission" work extends BOLA-118 (Keto-direct); do NOT revive the Kafka sync. See [[project_bola_auth_mode_deployment]] (that's AUTH_MODE/authentication — separate from this authZ/PERMISSION_PROVIDER decision). Ops-visibility [[project_bola_ops_visibility_ccs1]] is unaffected (reads BOLA's own DB).
