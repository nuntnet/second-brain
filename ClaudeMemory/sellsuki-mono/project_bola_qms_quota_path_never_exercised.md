---
name: project-bola-qms-quota-path-never-exercised
description: "BOLA→QMS reservation path has never run anywhere: BOLA dev+staging set QUOTA_ENFORCEMENT_DISABLED=true and no env has QMS quotas bola_workspace / bola_workspace_member; blocks OC-4631 AC-12 (found 2026-11-03)"
metadata:
  node_type: memory
  type: project
  originSessionId: 5b98ed42-0e86-44b4-98df-e4d0502d2574
  modified: 2026-11-03T08:45:02.954Z
---

Found 2026-11-03 while unblocking the OC2Plus × มหานครโลหะ prod path (CCS !418 HOLD):
- QMS main has the 4 reservation RPCs since 2026-09-29 (`2ef95ed`) — a deploy job, not dev. Prod QMS = v0.1.1.
- The CCS bind itself (BOLA CreateWorkspace with company_id) reserves `bola_workspace`, not only admin add.
- BOLA dev and staging run with `QUOTA_ENFORCEMENT_DISABLED=true`, so prod would be the first env to exercise it.
- No env has the two quota definitions or any plan for them; drafted payloads + runbook in
  `docs/release/qms/` (report-only, 1 workspace / 20 admins = default of D-292).
- rps v1.2.0 is on prod (deployed 2026-10-27) but whether migrations 0013–0035 ran there is unknown.

- Seat quotas: QMS renew/re-assign RESET to the starting balance (not +1) — never set active_until, never re-assign. Report mode drifted up (over-limit not debited, release credits) → payload rev 2 debits below zero; CreateQuota always writes rev 1 so the fixed revision differs per env (staging 2, fresh 1).

**Why:** the earlier analysis said "QMS code may not exist" and "fails at admin add"; both were wrong.
**How to apply:** order = QMS main → staging → create quotas + assign on staging → turn BOLA enforcement on
in staging and test bind/admin/release → same on prod (user runs) → Vault BOLA_SYSTEM_TOKEN → un-draft CCS !418.
Related: [[reference-prod-port-forward-denied-kubectl-get-logs-ok]].
