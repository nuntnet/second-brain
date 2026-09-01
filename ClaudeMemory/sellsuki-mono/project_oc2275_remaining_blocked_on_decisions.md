---
name: project-oc2275-remaining-blocked-on-decisions
description: "OC-2275's unbuilt scopes are blocked by product/architecture decisions, not by unwritten code — member.read and the reserved names were the only buildable ones"
metadata:
  node_type: memory
  type: project
---

Of the four API-key scopes the OC2Plus key picker greys out, only
`member.read` (OC-4468) could be built without a decision first. Verified
2026-09-01 while trying to finish the epic:

- **OC-4471 `point.adjust`** — blocked by **OC-4294**, which owns the first
  `AdjustPoint` use case in the system. `git grep AdjustPoint` across all three
  CRM services returns nothing, so building the API-key entry point first
  produces the two parallel point-adjust implementations the epic forbids.
- **OC-4470 `point.redeem`** — the v2 entry point is a small port of
  `CampaignRedemptionConfirm`, but the card requires idempotency and an atomic
  non-negative deduction, and **neither exists** in the shared point engine that
  also serves the customer LINE flow. That is a change to a live path plus a
  schema constraint.
- **OC-4469 `member.manage`** — dedup of a partner-created member against a LINE
  identity is unsettled per its own card.
- **OC-4475** (stamp `api_key_id` on point transactions) needs a migration, and
  migrations reach the CRM DB only by hand — see
  [[project-oc2275-crm-migrations-run-by-hand]].

**How to apply:** don't plan these as "just code". Each needs its decision or
dependency closed first, and OC-4471 in particular will silently create
duplicate logic if picked up before OC-4294. Landed instead: OC-4468 + OC-4429
(reserved scope names) in 3rdparty-api !219, frontend flag in !561 — which must
merge only AFTER !219 deploys, or the picker offers a scope that 403s.
