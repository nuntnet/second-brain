---
name: project_oc3559_member_tier_state
description: "OC-3559 member tier — what is built on which branch, the three things still genuinely blocked, and the 2026-09-13 benefits-field scope addition"
metadata: 
  node_type: memory
  type: project
  originSessionId: 1794f9b5-56ea-4edf-b5b3-a11df81d7cb5
  modified: 2026-09-13T15:30:23.216Z
---

## 2026-09-14 — 016 + 020 apply บน dev-th แล้ว เหลือแค่ permission

CRM DB `development_oc2plus_crm` (staging-th cluster, ns `octoplus-dev`) รัน
**016 + 020 ใน transaction เดียว** แล้ว → 4 ตาราง + 11 index + `benefits jsonb
NOT NULL DEFAULT '[]'` · รันซ้ำได้จริง (exit 0, NOTICE skip 12, ERROR 0)

`GET /v1/company/{id}/point/{pointId}/member-tier` บน dev-th **ตอบ 200 แล้ว**
(เดิม 500 เพราะไม่มีตาราง) — gateway rewrite `/backoffice/v1` → `/v1` ยิงตรงที่ pod
ต้องใช้ `/v1`

**GET ไม่ได้เช็ค membertier.manage** — มันเช็ค `oc2plus.member.view` ที่ Company Owner
มีอยู่แล้ว ส่วน **write ทุกตัวเช็ค `oc2plus.membertier.manage`** ซึ่งบน dev-th ยัง
**0 Keto tuple และไม่มีในแคตตาล็อก rps** → อ่านได้ เขียนไม่ได้

วิธีรัน DDL บน dev-th โดยรหัสผ่านไม่ออกจากคลัสเตอร์: pod `postgres:16-alpine` ที่
`sleep 3600` + env จาก `secretKeyRef` ของ secret `oc2plus-crm-secret` + configmap
ที่ใส่ไฟล์ .sql แล้ว `psql -v ON_ERROR_STOP=1 --single-transaction -f ... -f ...`

⚠️ **classifier ของ harness บล็อก grpcurl ที่เป็น write** (CreatePermission) — อ่านได้
เขียนไม่ได้ ต้องให้ user รันเอง ดู [[reference_harness_classifier_blocks_secrets_and_mutations]]

**แคตตาล็อก rps ≠ Keto grant** — สองอย่างนี้ไม่ผูกกันเลย พิสูจน์บน dev-th:
`oc2plus.pointclaim.review` **อยู่**ในแคตตาล็อก (สร้าง 2026-09-11) แต่ **0 tuple** ·
`oc2plus.news.manage` **ไม่อยู่**ในแคตตาล็อก แต่มี **500+ tuple** · แถวแคตตาล็อกคือสิ่งที่
ทำให้ permission ไป "โผล่ในหน้าแก้ role ของ CCS" ส่วน tuple คือสิ่งที่ทำให้ enforcement ผ่าน

`CreatePermission` gRPC **ไม่ต้อง auth** (handler ส่ง identity เปล่า) และ **ไม่ใช่ upsert**
— GORM `Create` ธรรมดา รันซ้ำได้ `Internal: database error` ซึ่งแปลว่ามีแถวอยู่แล้ว

## 2026-09-13 — PO added a new field mid-flight, on purpose, despite card already Ready to test

User (PO) asked for a per-tier free-text "สิทธิประโยชน์" (benefits) list — what a
tier gives beyond points/discount, e.g. "ของขวัญวันเกิด", "เมนูพิเศษเฉพาะสมาชิก" —
referencing a competitor's LINE member LIFF (Maguro) where each tier card shows a
spend threshold + a bullet list of perks. None of the existing fields
(`earn_multiplier`, `discount_percent`, `qualify_points`) cover this — it's pure
marketing copy, not something the engine reads.

I flagged that all 3 implementation branches (member-api tier engine,
backoffice-api CRUD, FE admin page) were already **Ready to test (DEV)**, and per
[[feedback_no_scope_change_in_sprint]] recommended a new follow-up card instead
of editing OC-3559 in place. **User explicitly chose to edit OC-3559 directly
anyway** — a deliberate one-off override of that standing rule, not a reversal of
it; don't assume future advanced-status cards get the same treatment without
asking again.

Added directly to OC-3559's description (full-rewrite via `editJiraIssue` +
`contentFormat: markdown`, ~28k→~30k chars, still under the 32,767 cap — see
[[reference_jira_editissue_adf_breakage]]):
- New PO-decision callout dated 2026-09-13 (parallel to the existing 2026-09-11 one)
- **L13**: `benefits` is pure marketing text, not read by any logic (unlike
  `earn_multiplier`/`discount_percent`) — ordered list per tier, not one blob
- **S1**: `member_tier.benefits` (jsonb, ordered array of short text, default `[]`)
- Flow A step 4, backoffice UI bullet, customer app `ovBenefits` UI bullets
  (must render the FULL list, not truncate to design's 2-line mock)
- **AC21** + QA test row 22

This means: whichever branch/dev picks this back up needs to add the `benefits`
column + CRUD + FE list-editor + customer app rendering on top of what's already
built — it's real incremental work, not just a description change.

**สถานะ branch (แก้ 2026-09-14):** งาน OC-3559 ชุดแรก **merge เข้า develop แล้วทั้ง 3 repo** — backoffice-api image ที่ deploy บน dev-th (`4faf5f31`) มี member_tier ครบ และ rps image (`650250da`) คือ commit แคตตาล็อก `4bcd8a2` เอง · ส่วน **benefits (scope change) ยังอยู่บน 3 branch `feat/oc-3559-tier-benefits` ที่ยังไม่ push** · ย่อหน้าด้านล่างนี้เขียนไว้ตอนยังไม่ merge อ่านเป็นประวัติ

**1. member-api `feat/oc-3559-tier`** (worktree `.worktrees/oc-3559`, commit `a06cbec`)
— pre-existing, NOT written by me. Owns `migrations/016_create_member_tier.{up,down}.sql`
(4 tables) + a **read-only** repository + `GetMyTier` / `GET /me/tier`. Its own
repository comment says the writer "belongs to backoffice-api".

⚠️ **This is invisible to `grep -i tier` on `src/`** — it is under `.worktrees/`.
OC-3559's own SUPPORT table claims "member-api = 0 hits" for exactly this reason,
and I nearly wrote a duplicate `016_*.sql` before finding it. Two files with the
same number collide in `migrate-all.sh`'s sort-order apply.
See [[reference_parallel_sessions_duplicate_symbols]].

**3. FE `feat/oc-3559-tier-admin`** (worktree `frontend/oc2plus-linecrm-frontend-backoffice/.worktrees/oc-3559`,
1 commit) — หน้า `เครื่องมือ > ระดับสมาชิก` + tier panel บน Member Detail + แก้
`MemberDetail.vue:654` ที่ hardcode `memberTier=""` (การ์ดบอก :648 — drift แล้ว)
**verify ในเบราว์เซอร์จริงแล้ว** ต่อ API ของ branch นี้บน DB จริง: เมนู, หน้า, point-unit
selector, empty state, `/member/{id}/tier` + `/tier/history` = 200, history เรนเดอร์ชื่อชั้น
เหตุผล วันที่ พ.ศ. และ note ของ manual override

⚠️ **RPS/Keto ที่เครื่องนี้ไม่มี tuple ของ company `11111111-…`** → `membertier.manage` = false
หน้าจึงขึ้นโหมด read-only (ถูกต้อง) · จะทดสอบ manage path ในเบราว์เซอร์ต้อง grant สิทธิ์ก่อน
ตอนนี้คุมด้วย component test แทน

**2. backoffice-api `feat/oc-3559-tier-admin`** (worktree `.worktrees/oc-3559-tier-admin`,
4 commits) — the write side: entity ladder validation (AC1/AC15), repository,
program+ladder CRUD, `EvaluateMemberTierOnEarn` (AC2/3/4/14), `SweepExpiredTiers`
(AC5/AC6), override (AC13), 8 HTTP endpoints, `cmd/tier_sweep` with `-dry-run`.
Plus rps `2d15f96` seeding the two permission catalog rows (รันจริงแล้ว idempotent).

**เจอตอน verify ด้วยตา ไม่ใช่ตอนอ่านโค้ด:** history เรนเดอร์ actor เป็น uuid ดิบ →
แก้โดย resolve เป็นชื่อฝั่ง server ด้วย `resolveCreatedByName` (ตัวเดียวกับที่
`adjusted_by_name` ของ OC-4294 ใช้) แล้ว FE fallback เป็น `แอดมิน · <8 ตัวแรก>`

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
