---
name: project_oc4362_approve_and_admin_edit
description: OC-4362 receipt-claim approve foundation lives on local/approve-plus-ocr (base-rate award); admin-edit §B built on top
metadata: 
  node_type: memory
  type: project
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-09-07T11:50:48.481Z
---

**The approve→award foundation for OC-4362 EXISTS but is unmerged.** As of
2026-09-07, backoffice-api branch **`local/approve-plus-ocr`** (a local integration
of OC-4362 approve + OC-4464 OCR providers) has `src/use_case/point_claim_approve.go`
— `ApprovePointClaim` + `ApproveAndAward` (one atomic txn, concurrency-guarded) that
awards **base-rate** points = `computeBaseRatePoints(claim.PurchaseAmount, baseRate)`
(OC-4415 base rate). It is NOT merged to develop, and it is base-rate only — the full
per-product award engine (OC-4413/4420) is still green-field / not in any tree.
The working tree default was `feat/oc-4421-product-scope-core` (only `product_scope.go`,
no approve). Correcting an earlier wrong claim that "approve doesn't exist".

**Award reads PurchaseAmount from the row inside ApproveAndAward** → so editing the
amount on a pending claim before approve changes the awarded points automatically.
That is the hook for OC-4362 §B.

**§B admin-edit-before-approve — DONE + verified 2026-09-07, committed (NOT merged):**
- backoffice-api branch `feat/oc-4362-admin-edit-claim` (off approve-plus-ocr),
  commit `90a5804`: `POST /company/{id}/point-claims/{id}/edit {purchase_amount?,
  purchase_date?}` → `EditPendingPointClaim` → repo `UpdatePendingFields` = one atomic
  UPDATE guarded by `company_id` AND `status='pending'` (0 rows → CLAIM_NOT_FOUND);
  validates amount>0 / non-future date; before→after in `slog.Event`,
  `slog.Audit(AuditActionUpdate)` (mirrors RejectPointClaim). Errors 400
  VALIDATION_FAILED / 404 CLAIM_NOT_FOUND. use_case cov 87.7%.
- backoffice frontend branch `feat/oc4362-approve-button`, commit `0ddd224`:
  pending-only "แก้ไข" toggle → amount/date inputs, keeps member original visible,
  hint "ยอดนี้จะถูกใช้คำนวณแต้มเมื่ออนุมัติ"; 400 inline / 404 stale-refetch / toast.
  (Same commit also lands the earlier-approved 3-column review modal + filter polish.)
- Verified end-to-end in the admin UI (:5176) + HTTP: #999 999→300 (UI), #123456
  1000→250 (curl); both persist as pending; DB confirmed.

**Env changes made this session (to revert if needed):** switched backoffice-api
submodule branch `feat/oc-4421-product-scope-core` → `feat/oc-4362-admin-edit-claim`;
`git stash`ed backoffice-api `cmd/generics_server/main.go` (local OCR provider config —
paddle/gemini). Monorepo submodule refs NOT bumped (stacked on unmerged branches).

**§A OCR cross-check gate (OC-4464) — DONE + verified 2026-09-07:**
- backend `feat/oc-4362-admin-edit-claim` commit `9a7087d`: `ApprovePointClaim` gained
  `acknowledgeReceiptMismatch bool`; before the base-rate lookup it reads the OCR result
  (`pointClaimOCRRepository.GetByClaim`) and if `compare_result.source_reference ==
  "mismatch"` and not acknowledged → `ErrPointClaimReceiptMismatch` → **422
  `OCR_RECEIPT_MISMATCH`**. OCR absent/errored/unreadable never blocks (pure helper
  `receiptNumberMismatchBlocksApprove`, 7 unit cases). Approve body is optional
  `{acknowledge_receipt_mismatch?: bool}`.
- frontend `feat/oc4362-approve-button` commit `e161eeb`: flipped `APPROVE_ENABLED=true`;
  approve-confirm modal shows a red warning when `compareResult.source_reference==='mismatch'`
  and sends the ack on confirm; defensive catch of `OCR_RECEIPT_MISMATCH`. Verified in
  the admin UI (:5176): Shopee #2408… mismatch → warning → confirm → DB approved.
- The OCR per-field compare (`compare_result`, `mismatch_count`) already existed +
  persisted; §A only turned the receipt-number mismatch into a gate. §A phase-2
  (per-company hard-block-vs-warn config) NOT done — MVP is "must acknowledge".
- **Test-data note:** approving during verification moved claims 24539127/82dd758d/
  46afad3d/2408SHOPEE1788705310 pending→approved (awards written, not reversed).

**§C image-hash dedup (OC-4362) — DONE + verified 2026-09-07:**
- member-api `feat/oc-4407-marketplace-claim` commit `83cff74`: `point_claim.image_hash`
  varchar(64) = SHA256 hex of the uploaded bytes (migration **010**, APPLIED locally);
  partial unique index `crm_point_claim_company_image_hash_uidx (company_id, image_hash)
  WHERE status IN ('pending','approved')` mirrors the order_ref index. Both submit paths
  (`SubmitPointClaim` + `SubmitMarketplaceClaim`, shared `pointClaimImageHash` helper) set
  it; `CreateSubmission` maps the constraint → `ErrPointClaimDuplicateImage` → **409
  `DUPLICATE_IMAGE`**, handled like DUPLICATE_RECEIPT (photo orphaned).
- member FE `feat/oc-4462-my-claims` commit `75cc83c`: DUPLICATE_IMAGE → inline error
  near photo, i18n "รูปใบเสร็จนี้เคยส่งเข้ามาแล้ว".
- **Verified HTTP:** submit image → 200; same image + DIFFERENT receipt number → 409
  DUPLICATE_IMAGE (order_ref would have missed it). Exact-hash MVP; **perceptual hash
  (re-photographed receipt) is phase-2** (needs image lib + Hamming search).
- Submit rate-limit = **5/member/24h** (DB `CountByMemberSince`, not Redis). Migration
  010 must be applied wherever this deploys (run-by-hand CRM schema).

**award_dedup_registry wiring (OC-4420) — DONE + verified 2026-09-07:**
- backoffice-api `feat/oc-4362-admin-edit-claim` commits `d8017df` + test `07149ad`.
  The table (migration 008) shipped dormant — no code touched it. Wired into
  `ApproveAndAward`/`awardWithinTx` (`point_claim_repository/postgresql.go`): the approve
  UPDATE now RETURNs `order_ref`+`channel`; inside the same txn a registry row
  `(company_id, order_ref, member_id, channel, result-jsonb)` is inserted after the award.
  A 23505 on `award_dedup_registry_company_order_ref_uidx` → `isUniqueViolation` →
  `ErrPointClaimAlreadyAwarded` → **409 ALREADY_AWARDED**, whole txn rolls back.
- NOTE: `awardWithinTx` on this branch ALREADY back-fills `point_claim.member_point_activity_id`
  (my earlier "never written" scope note was from the pre-approve OC-4421 tree).
- Verified: approve INV-DEDUP-AAA → registry row `receipt:INV-DEDUP-AAA | manual |
  {"points":14,...}`, claim linked; a duplicate (company, order_ref) insert is rejected by
  the unique index. In the current receipt/marketplace-only world the submit-side partial
  index already prevents a second claim per order_ref, so the registry violation is a
  defensive/cross-transport guard (POS/Kafka future) — its concrete value now is making the
  table live + the result snapshot.

**OC-4464 §B OCR line-items (capture + admin-visual pick) — DONE + committed, LIVE VERIFY PENDING (2026-09-07):**
- member-api migration `712285d`: `point_claim_ocr_result.read_items jsonb` (011, APPLIED locally).
- backoffice-api `c1f301a` (branch feat/oc-4362-admin-edit-claim): `PointClaimOCRLineItem`
  {Name,UnitPrice,Quantity,LineAmount} on the OCR read model; iApp maps
  `processed.items[].{itemName,itemUnitCost}` (VLM/paddle best-effort); persisted+scanned;
  ocr-result response carries `items`. **Privacy: issuer name/tax id still NOT captured**,
  only item name+price (iApp leak test updated). 1309 tests pass.
- backoffice FE `d2c4f18` (branch feat/oc4362-approve-button): OCR column shows the items
  table; pending-claim checkboxes → selected-sum (exact BigInt decimal math) → button
  "ตั้งเป็นยอดซื้อ" sets the §B edit amount (no auto-save). 43/43 tests.
- **per-brand = admin picks the brand's items by eye (PIS auto-match is the LATER piece 3).**
- ⚠️ **Live browser proof NOT done** — the local OC2Plus stack (:8102/:8089/:5176/:5183) CRASHED
  mid-verify. Seeded OCR items on claim `LOCAL-NOLINE-1788691244` (bc254b1b): 2 โฟร์โมสต์
  ฿15+฿15 + 2 others, ready to test once the stack is back.

⚠️ **OPERATIONAL LESSON (2026-09-07):** dispatching **two heavy `developer` agents in parallel**
(each running `go build ./...` + `go run generate_fiber_interface` codegen + `vitest run`)
while the local stack runs under overmind/air **crashed the whole OC2Plus overmind session**
(member-api/backoffice-api/both frontends went down together; postgres/redis survived; no root
`.overmind.sock`). Ports don't auto-recover. Don't run 2+ build-heavy agents concurrently against
a live local stack — serialize them, or expect to restart the stack. See [[project_overmind_restart_quirk]].

**Per-brand epic completion — planned as 4 NEW cards under OC-2743 (2026-09-07):**
- **OC-4480** [Spike] lock 5 decisions (brand-source, match method+threshold, customer amount field,
  OCR reliability bar, award engine) — blocks the other 3.
- **OC-4481** auto-match OCR items ↔ PIS/resolved_skus + pre-tick in review (piece 3) — blocked by
  OC-4480 + OC-4421; depends on OC-4464 §B (done).
- **OC-4482** member submit: derive/optional purchase_amount for per-brand campaigns.
- **OC-4483** customer claim result: per-item earned/not-earned breakdown.
Dependencies are in each card's description (tables); formal issue-links NOT yet created.

**⚠️ OCR real-image proof BLOCKED on provider creds:** backoffice-api `.env` has
`RECEIPT_OCR_VLM_BASE_URL=https://openrouter.ai/api/v1` but the key `AQ.Ab8RN…` is NOT an
OpenRouter key (`sk-or-`) → 401 → OCR job PROVIDER_ERROR. The vlm.go code is correct (fetched image,
asked for items, reached the provider). Fix = user edits .env (native-Gemini base URL to match the
Google-looking key, OR a real sk-or- key) + restart backoffice-api; then resubmit
`real-receipt-items.jpg` (in scratchpad, 5 lines incl 2 โฟร์โมสต์). Provider=vlm, worker ticks ~30s,
5/member/24h submit cap (DB CountByMemberSince — age claims to reset).

**Services I hand-started (outside overmind, user's root overmind session died):** backoffice-api
:8089 (air), backoffice FE :5176 (bun), consent :8096 (air), member-api :8102 (air). rps :9998 /
keto :4466 / mongo / kafka / redis survived. To restore normal setup: kill these + `make stop` then
`make dev`. See [[project_overmind_restart_quirk]].

Remaining OC-2743 loyalty concerns:
per-brand **auto-matching** receipt items ↔ PIS/`resolved_skus` (OC-4481, carded) + the full
campaign award engine (not base-rate) — the big automation. See
[[project_oc4362_claim_cluster_gaps]] [[project_loyalty_point_cluster]]
[[reference_daisyui_progress_class_collision]].
