---
name: project_oc3559_member_tier_state
description: OC-3559 member tier — what is built on which branch, and the three things still genuinely blocked (2026-09-12)
metadata:
  type: project
---

**Tier work lives on TWO unmerged branches in different repos. Neither is on develop.**

**1. member-api `feat/oc-3559-tier`** (worktree `.worktrees/oc-3559`, commit `a06cbec`)
— pre-existing, NOT written by me. Owns `migrations/016_create_member_tier.{up,down}.sql`
(4 tables) + a **read-only** repository + `GetMyTier` / `GET /me/tier`. Its own
repository comment says the writer "belongs to backoffice-api".

⚠️ **This is invisible to `grep -i tier` on `src/`** — it is under `.worktrees/`.
OC-3559's own SUPPORT table claims "member-api = 0 hits" for exactly this reason,
and I nearly wrote a duplicate `016_*.sql` before finding it. Two files with the
same number collide in `migrate-all.sh`'s sort-order apply.
See [[reference_parallel_sessions_duplicate_symbols]].

**2. backoffice-api `feat/oc-3559-tier-admin`** (worktree `.worktrees/oc-3559-tier-admin`,
4 commits) — the write side: entity ladder validation (AC1/AC15), repository,
program+ladder CRUD, `EvaluateMemberTierOnEarn` (AC2/3/4/14), `SweepExpiredTiers`
(AC5/AC6), override (AC13), 8 HTTP endpoints, `cmd/tier_sweep` with `-dry-run`.
Plus rps `2d15f96` seeding the two permission catalog rows.

## Card facts that are WRONG — report, don't follow
- Permission prefix: card says `sellsuki.oc2plus.membertier.manage`. Every real
  OC2Plus code is `oc2plus.*`. Correct: `oc2plus.membertier.manage` / `.override`.
- `MemberDetail.vue:648` is actually **:654**.
- `member_tier*.point_unit_id` is `text` while the rest of the CRM schema uses
  `point_id uuid` — comparisons rely on an implicit cast and blow up on a
  non-uuid value.

## Still genuinely blocked (do NOT guess these)
- **AC9/AC16/AC17 — multiplier in the award engine.** Two reasons, both hard:
  open questions 3 (rounding) and 4 (tier × campaign stacking) are undecided and
  change real point amounts; AND the engine has no tier input at all
  (`3rdparty-api award/types.go` `MemberFacts` has `SegmentIDs`, always nil, and
  no `TierID`), with OC-4413 unmerged. `earn_multiplier` is stored and served
  only — nothing multiplies by it. `computePoints` (`award/evaluate.go:307`) is
  where it would go; `roundDiv` already exists with floor/ceil/nearest.
- **AC19/AC20 — customer-app ovBenefits.** The design v3 `.dc.html` is not in the
  repo, and the card says AC20 needs design to add the expiry field first.
- **AC10/AC11/AC12 — BOLA sync.** Not started. Note `POST /v1/contacts/upsert` is
  **async** (202 + job_id) and guarded by `FlatAdminGuard`, needing
  `AUTH_MODE=header` + `INTERNAL_AUTH_SECRET` + `?workspace_id=`.

## v1 boundaries taken from the card's own proposals
`qualify_basis` = `points_earned` only · `period_mode` = `fixed_period` only ·
`discount_percent` stored and displayed, never enforced. Both enums are closed in
code AND in the DB CHECK — widening either is a second engine, not a config value.

## Sequencing
backoffice-api's code cannot run until member-api `feat/oc-3559-tier` merges —
that branch owns the tables. Say so in the MR.

See [[project_oc2plus_tier_is_per_company_config]] · [[reference_goqu_dialect_blank_import]] ·
[[project_loyalty_canonical_contract]] · [[reference_oc2plus_schema_lives_in_external_repo]]
