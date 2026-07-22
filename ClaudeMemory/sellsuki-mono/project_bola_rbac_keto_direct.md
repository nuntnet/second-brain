---
name: project-bola-rbac-keto-direct
description: "BOLA SaaS authz = Keto-direct (PERMISSION_PROVIDER=keto), NOT the CCS→Kafka→local sync; 3-tier model; sync MRs superseded"
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-07-22T16:06:23.268Z
---

**Decision (CTO, 2026-07):** BOLA authorization uses the pluggable `AdminRBACRepository` seam (`src/use_case/repository/admin_rbac_repository.go`, providers `inline`/`local`/`keto`, wrapped in `NewCached`). 
- **SaaS mode → `PERMISSION_PROVIDER=keto`** (query Sellsuki shared Keto at runtime, cached 30s) — this is **BOLA-118** (was a stub `keto_stub.go`, `NewKetoStub()` at `cmd/bola_server/helper.go:665`).
- **Self-host → `PERMISSION_PROVIDER=local`** (BOLA-205, DB-backed, full: custom roles + per-resource). Untouched.

**Rejected: the CCS→Kafka→local-table sync.** The sync consumer wrote CCS membership events into BOLA's **local admin table** (`adminRepository.CreateAdmin/UpdateAdmin` in `membership_sync.go`) → that's `PERMISSION_PROVIDER=local` fed by sync, which does NOT satisfy "SaaS uses Keto." Doing sync then Keto-direct = duplicate + two-sources-of-truth drift (same class as [[project_bola_contact_profile_model]] BOLA-173). So sync is dead:
- **MR !138** (bola-backend consumer) — MERGED but dormant (`CCS_MEMBERSHIP_SYNC_ENABLED=false`, auto_push_worker not deployed anywhere; migration 0128 applied harmlessly). Superseded → remove in a cleanup PR.
- **MR !251** (CCS3 producer) — CLOSED.

**3-tier authz model — BOLA owns `bola_workspace` Keto namespace DESIGN + READS only:**
- system admin (ops, all workspaces) = `BolaInstance:sellsuki#system_admin`
- company admin (company-bound workspaces) = `BolaCompany:{cid}#admin` via workspace→company link
- workspace super_admin (own workspace) = `BolaWorkspace:{ws}#super_admin`; + admin/editor/viewer
- Custom roles + per-resource = local only, unsupported in keto mode.

**⚠️ Model correction (2026-07-22) — Sellsuki Keto is FLAT, not OPL:** `scripts/keto.yml` + role-permission-service CLAUDE.md show fixed namespaces `roles`/`permissions`/`role_lists`/`contexts`/`identities` — **no OPL `implements Namespace`/`permits` subject-set rewrite, no per-domain namespaces**. It's a hybrid: role presets in PostgreSQL (role-permission-service) → assign expands to flat tuples (`roles`: Object=`bola.workspace:{ws}` Relation=`<role>` Subject=user; `permissions`: Object=`<perm>` Relation=`bola.workspace:{ws}` Subject=user) → check = single tuple lookup in `permissions` ns. So BOLA does NOT create a Keto namespace — it reuses `roles`/`permissions` + a PostgreSQL role preset. The earlier `bola_opl.ts` (`class BolaWorkspace implements Namespace`) design was WRONG and withdrawn. **3 tiers are realized in the CHECK layer** (BOLA `CheckPermission` does up to 3 exact-match checks — `bola.workspace:{ws}` → `bola.company:{cid}` → `bola.instance:sellsuki` — OR'd), NOT via OPL traversal. **New cross-cutting dependency:** shared `entity` package (`.../backend/entity`, only knows `sellsuki.*`/`patona.*`) must register kind `bola.workspace`(+`bola.instance`) or `access_control.Namespace`/`AssignRole` validation fails.

**Direction B decided (2026-07-22) — role-permission-service writes tuples, NOT BOLA:** BOLA delegates ALL bola-workspace Keto writes to the shared `role-permission-service` via its invite/`AssignRole`/`UnassignRole` flow (tenant refs are arbitrary strings — `company:X`, `store:Y`, `bola.workspace:Z` — confirmed `role_value_object.go`). BOLA only: designs the OPL namespace, calls role-permission-service on role change, and READS Keto to enforce. (Supersedes the earlier "BOLA writes all tuples itself" idea.)

**Invite flow follows from B (BOLA-293, blocked-by BOLA-118):** BOLA kratos-mode invite currently emails a **Kratos recovery link** (`admin.go` GenerateRecoveryLink → `/admin/recovery/code`) → lands on the shared `/recovery` "ลืมรหัสผ่าน" page (kratos-ui-go) — a pre-Keto **stopgap** (F8 `AllowNoRole=true`, metadata-only). Target: reuse the **platform invite flow** `join.sellsuki.com/invite/<code>` (frontend/sellsuki-invitation SPA + role-permission-service `CreateInvite`/`AcceptInvitation`) which already shows role + Keto permissions + Accept/Decline. Needs bola-workspace roles in Keto (BOLA-118) + SPA generalized company→workspace + post-accept BOLA local admin row.

**Why:** dual-mode provider already satisfies both requirements with one code path, zero sync infra. BOLA roles today are invite Metadata only (`AllowNoRole=true`, F8 shipped) — no bola role/permission tuples in Keto yet. Fix = add bola-workspace role presets in role-permission-service (PostgreSQL) + register the `bola.workspace` kind in the shared entity package (see model correction below). No new Keto namespace.

**How to apply:** any future "BOLA SaaS permission" work extends BOLA-118 (Keto-direct); do NOT revive the Kafka sync. See [[project_bola_auth_mode_deployment]] (that's AUTH_MODE/authentication — separate from this authZ/PERMISSION_PROVIDER decision). Ops-visibility [[project_bola_ops_visibility_ccs1]] is unaffected (reads BOLA's own DB).
