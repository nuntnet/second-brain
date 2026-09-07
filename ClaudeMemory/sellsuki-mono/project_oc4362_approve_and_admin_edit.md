---
name: project_oc4362_approve_and_admin_edit
description: OC-4362 receipt-claim approve foundation lives on local/approve-plus-ocr (base-rate award); admin-edit §B built on top
metadata: 
  node_type: memory
  type: project
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-09-07T05:39:30.444Z
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

Remaining OC-2743 loyalty concerns (scope-extension comments posted on the cards):
§C image-hash dedup + wire `award_dedup_registry` (OC-4362/4420), OCR **capture items[]**
(OC-4464 §B), per-brand matching (OC-4413). See
[[project_oc4362_claim_cluster_gaps]] [[project_loyalty_point_cluster]]
[[reference_daisyui_progress_class_collision]].
