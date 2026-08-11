---
name: project-bola-saas-access-model
description: "BOLA SaaS access model decided 2026-08-11 (BOLA-309): layered — CCS3 owns company/org scope, BOLA owns inside-workspace scope, Kratos owns identity, Keto enforces; two earlier contradictory designs were both implemented"
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T15:12:28.134Z
---

**Decision (2026-08-11, epic BOLA-309):** BOLA SaaS access management is **layered by scope, not by product**:

| Layer | Owner | Actions |
|---|---|---|
| company / org | **CCS3** (`admin.sellsuki.com`) | create/rename/delete BOLA workspace within plan quota; invite a NEW person (no Kratos identity yet); decide which workspaces they may enter; remove from workspace |
| inside a workspace | **BOLA** | workspace super_admin (`admin.manage`) changes roles, adds/removes people who already have org access, workspace settings |
| identity / password | **Kratos only** | no password UI in BOLA or CCS3 on SaaS; reachable from BOLA My Profile + Team Members row |
| enforcement | **Keto** via rps presets | BOLA reads (BOLA-118 merged), rps writes all tuples |

**A workspace-scope invitee gets NO CCS3 access** — this deliberately kills BOLA-196's premise ("workspace member must be a company member first").

self-host (`CompanyID == ""` / `AUTH_MODE=local_jwt`) is untouched everywhere; every card in the epic carries a "self-host unchanged" AC.

**Why the epic exists — the failure worth remembering:** three contradictory answers to "who owns BOLA membership in SaaS" existed at once, **two of them shipped**, because neither design doc was ever marked superseded:
- `docs/bola-authz-ccs-sync.md` — CCS3 owns it, BOLA never calls Keto, Kafka sync (PROPOSAL, partly built as BOLA-285/286)
- `docs/bola-keto-rbac-implementation-spec.md` — BOLA reads Keto directly and BOLA issues the invite via rps (built as BOLA-118/293)
- Result: BOLA-286 hides Invite/Edit/Remove exactly in the mode where BOLA-293's invite lives → **SaaS invite has no reachable UI at all**, and "Manage in CCS" points at CCS3's *company* members page (`/users`) which is a different object. CCS3 has never had a BOLA workspace member page, and no card asked for one.

**Cards created:** BOLA-309 (epic, holds the persona journey matrix) → BOLA-310 CCS3 workspace CRUD+quota · BOLA-311 CCS3 invite (role × workspace, no CCS3 access) · BOLA-312 CCS3 add/remove existing user · BOLA-313 fix BOLA-286 gating to layered · BOLA-314 Kratos password deep-links + close the 3 unguarded password writes · BOLA-315 membership write contract + reconcile · BOLA-316 doc/backlog hygiene.

**Hard blocker before flipping `PERMISSION_PROVIDER=keto`:** BOLA-315 — `RemoveAdmin` deletes only the local row and never unassigns the Keto tuple, so "removed" users stay authorized the moment enforcement moves to Keto. `UpdateAdmin`'s role sync is best-effort and fails silently.

**How to apply:** read BOLA-309 before touching authorization/membership in bola-backend, CCS3, or rps; if two design docs disagree, stop and ask rather than picking one. See [[reference-rps-dual-mainline]] and [[project-bola-rbac-keto-direct]].
