---
name: project-oc4415-base-rate-state
description: OC-4415 base earn rate — config CRUD merged to main+develop, but always-on system campaign NOT built and entity NOT bumped (local composition)
metadata:
  type: project
---

OC-4415 "Base Earn Rate" — verified 2026-08-31 against origin (not a stale
working tree — see [[reference_grep_stale_branch_not_origin]]).

**Merged, on both `main` and `develop`** (backoffice-api) + frontend merged to
develop (branch `feature/oc-4415-base-earn-rate`, ahead 0):
- base rate CRUD (list/detail/create/update), `PointBaseRate` struct
- validation, rate-history audit endpoint, Keto server-side
  (`PermissionOc2PlusPointManage` — reused, not a dedicated `manage_base_rate`)

**Two deviations from the card the PO should know:**
1. **entity NOT bumped.** Card said "add 5 fields to `entity.point.Point`".
   They kept `entity` at v1.9.7 and composed `PointBaseRate` as a
   use_case-level struct in backoffice-api instead (deliberate, commented). Sound
   — 3rdparty doesn't need it — but if OC-4413 award engine must read base rate
   it can't get it from the shared entity; needs another contract.
2. **AC-03/Rule 2 `SYSTEM_BASE_EARN_{unit}` system campaign NOT seeded**
   (grep on main = empty). So base rate is stored/validated/audited/displayed
   but does NOT award points yet — the always-on core (AC-04/05/06) is deferred
   to OC-4413 award engine, which is still To Do. The card is NOT closed per AC.

**Branch divergence:** OC2Plus convention is MRs→develop only, but 4415 went to
`main` too. Both carry base rate; main ahead of develop by 2, develop ahead by 8.
Disabled rate stores NULL (not 0) so it can't read back as "free points every 0
baht". Blocks OC-4295 (campaign wizard) and OC-4413 (award engine), both To Do.
