---
name: project_oc4362_approve_and_admin_edit
description: OC-4362 receipt-claim approve foundation lives on local/approve-plus-ocr (base-rate award); admin-edit §B built on top
metadata: 
  node_type: memory
  type: project
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-09-07T16:43:18.607Z
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

**Phase-3 BACKLOG CLOSED 2026-09-07 late (everything that was "flag as backlog" and actually fixable):**
- member-api `4a9f61b` (!98): audit line on every session mint (LineLogin + OTPLogin), entity
  `oc2plus.crm.session`, refs `[sess.ID, line_login|otp_login]`, ids only.
- backoffice-api `2cd8897` (gated branch): award_dedup_registry row written even on a 0-point approval —
  `registerAwardWithinTx` split out of `awardWithinTx`, 409 handling covers both paths, snapshot points=0.
- backoffice FE `67c6ce5` (!572): `sumDecimalStrings` normalises grouping commas (`1,234.00` was silently
  dropped), strict plain-decimal regex, `sumDecimalStringsDetailed{sum, skipped}`; spec 20/20.
- member FE `dcf2e30` (!35): last DaisyUI bare-class collisions (`.label`/`.btn` pinned, `.toast`→
  `.member-toast`) in LoginView/MembershipView/IdentityModal — found by intersecting scoped selectors
  with daisyui/dist/styled.css; `member_not_found` strict on error_code, bare 404 → `not_found` copy, no
  register CTA.
- B1 rejected-card branch VERIFIED with real data: rejected `IC0626-0001` (member 06e811bd = the local
  session's member) → badge + meaning line + admin reason + "แจ้งใบเสร็จใบใหม่ →" button render.
  `INV-LOCAL-0002` (member 44444444, the 1×1-PNG claim) also rejected as test data — it is NOT this
  session's member, which is why it never showed in :5183.
- Jira status comments posted (not transitions — user decides status): OC-4362 #44546, OC-4464 #44547,
  OC-4348 #44548 (that card is still `To Do` with code in MRs — flagged for PO).
Still deliberately NOT done: DS-level a11y (ssk-modal role=dialog, ssk-input aria-invalid), systemic
hardcoded px, submit rate-limit TOCTOU, §A phase-2 per-company hard-block config, Phase-2 marketplace OCR.

**!538 (approve base-rate → develop) READINESS, checked 2026-09-07 late:** `has_conflicts: false`,
`merge_status: can_be_merged`, `blocking_discussions_resolved: true`, no approvers required. The ONLY
blocker is `detailed_merge_status: ci_must_pass` — its pipeline 57105 "failed" with
`stuck_or_timeout_failure`, `runner: null` = never ran (the `staging-th` runner fleet outage, see
[[reference_dead_staging_runner_tag]]). Code proven green locally on the exact sha `f0016f2` in a
detached worktree: `go build ./...` OK, `go test ./src/...` 19 pkgs ok / 0 FAIL. Fresh MR pipeline
**57557** queued (auto-runs when a runner returns). My glab token shows `project_access: null` →
cannot arm merge-when-pipeline-succeeds; that is a maintainer's one click. Once !538 lands, the
gated backend branch `feat/oc-4362-admin-edit-claim` (8 commits) rebases onto develop in one step.

**UX/UI pass (B1/B2/B3) SHIPPED 2026-09-07 — all pushed, all MRs → develop:**
- **B1 member FE `9d0c5c8`** (MR !35): company logo+name header on all 3 claim screens, purpose strip
  (why here / what you get — mechanism only, member-api has NO earn-rate/points-preview endpoint),
  status meaning lines, rejected→resubmit CTA, empty-state CTA; brand theme via existing
  `useThemeStore` `--c-primary/--c-font` vars; fixed 4 more DaisyUI bare-class collisions
  (`.badge/.toast/.label/.btn` → prefixed/pinned). Rejected-row branch verified via DOM clone only.
- **B2 backoffice FE `2e90fe5`** (MR !572): OCR status panel (STATUS/WHY/WHAT-TO-DO) in new pure
  `src/entities/point-claim-ocr-status.ts` — prefers `failure_reason`, falls back to `error_code`
  substring match; bad_image → reject CTA + NO retry; provider_unavailable → ลองอ่านใหม่ (409/429 inline);
  misconfigured → แจ้งผู้ดูแล, no retry. Scan-first layout (photo | 4-col comparison `<table>`, renders
  with no OCR result). Real cursor pagination (stores cursor-that-produced-page-i → Previous works),
  20/page, total only on pending. Fixed latent v-else-if bug hiding the read table + DS slot
  `suffix`→`postfix`. 42 i18n keys.
- **B3 backoffice-api `baa3104`** (gated branch) + **member-api migration 012 `cbef55e`** (MR !98):
  `failure_reason` enum `bad_image|provider_unavailable|misconfigured|unknown` — sentinels
  `ErrOCRBadInput`/`ErrOCRMisconfigured` multi-%w-wrapped with `ErrOCRPermanent` in `classifyStatus`;
  pure `ocrFailureReason(code,cause)` (table-tested); stored on the row at fail time (provider status
  not kept), cleared on every re-queue; exposed as optional enum on ocr-result (omitted when absent).
  **LIVE-PROVEN:** re-ran the 1×1-PNG claim `0747d2b4` → `failed|PROVIDER_ERROR|bad_image`, API returns
  `"failure_reason":"bad_image"`. `nullableString` in this repo takes `*string` (compile trap).
- Local config files (`.env.development`, `vite.config.ts`, `bun.lock`, `.bak`) deliberately NOT
  committed in either FE — they carry local caddy/port overrides.

**Phase-3 review gates RUN + findings FIXED (2026-09-07):** ran /code-review + /security-review
(all 4 repos) + /design-review + /accessibility-review (2 FEs) via 6 opus subagents. NO 🔴 correctness/
tenant/keto blockers anywhere. Fixed + pushed:
- **member-api `4618e38`** (MR !98): 🔒 OTPLogin didn't consume the OTP → one code minted unlimited
  sessions for 5 min (test-mode = trivial replay). Now consumes after member-resolve: MarkUsedAtomically
  (GETDEL, concurrent gate) + persist UsedAt (checkOtpSessionState blocks sequential/test-mode replay).
  +replay-blocked test. Also fixed stale integration_id comment.
- **backoffice FE `37c2aed`** (MR !572): 🔴 date input no label → id+aria-label; 🔴 OCR checkbox no
  accessible name → aria-label on the NATIVE input (ssk-checkbox renders its input in shadow DOM with a
  broken internal `<label for>` and NO aria-name hook — host aria-label can't cross the boundary, so
  native+aria-label is the accessible choice, documented inline); role=alert on mismatch/errors + sr-only
  live mirror for ssk-input's shadow error; money right-aligned; visible focus ring.
- **member FE `e334429`** (MR !35): OTP autofill (`autocomplete=one-time-code`)+paste-distribute (was
  truncated to box 1); resend failure now surfaced on OTP screen + cooldown reverted; phone/OTP labels +
  role=group + aria-live error regions; DUPLICATE_IMAGE error cleared on photo re-pick.
- **backoffice-api `aae4ad7`** (gated branch): approve audit EntityRefs gains `ocr_mismatch_acknowledged`
  when admin overrides the §A gate (fraud trail).
Flagged as backlog (NOT fixed — DS-level / systemic / pre-existing): ssk-modal no role=dialog/aria-modal,
ssk-input no aria-invalid/describedby, hardcoded-px matches sibling convention, `.badge`/`.toast`/`.btn`
DaisyUI bare-class collisions in touched views, login has no audit trail (matches LineLogin), submit
rate-limit TOCTOU, award_dedup registry skipped at 0 points, VLM PII is prompt-only (iApp/paddle drop it
structurally).

**OCR full-pipeline proof CLOSED end-to-end with REAL Gemini (2026-09-07):**
submit (member-api 201) → OCR worker (backoffice-api) → native Gemini → `read_items`
populated. Claim `766a9b29` came back `job_status=partial`, `read_items`=5 lines
(2× นมโฟร์โมสต์ ฿15, ขนมปัง ฿22, โค้ก ฿35, มาม่า ฿7), `read_source_reference=RCPT-2026-778899`,
`read_purchase_amount=123.00`. Status is **partial (not succeeded) because §A receipt-number
cross-check FIRED live** — member typed `RCPT-FULLPIPE-02`, OCR read `RCPT-2026-778899` off the
image → mismatch. That is the gate working, a bonus proof, not a defect.

TWO REAL BUGS fixed in `receipt_ocr_repository/vlm.go` (commit **`24ca497`** on branch
`feat/oc-4362-admin-edit-claim`, tests green, monorepo ref NOT bumped — stacked on unmerged):
1. **MaxTokens:500 truncated thinking-model output** (the actual PROVIDER_ERROR).
   gemini-2.5-flash is a *reasoning* VLM: hidden reasoning tokens count against `max_tokens`.
   At 500 a real read spent ~476 tokens thinking → `finish_reason:"length"` → JSON cut at
   `{"source_reference":"RCPT-2026-778899` → `extractJSONObject` fails → `unparsable_extraction`
   → surfaced to admin as spurious PROVIDER_ERROR. Raised to **2048** → `finish_reason:"stop"`,
   full JSON. A non-thinking model is unaffected. THIS is why direct curl "worked" earlier: my
   ad-hoc curl used a short prompt/high budget; the worker's real 500 budget was the difference.
2. **vlm borrowed iApp's 20s `requestTimeout`** — added its own `vlmRequestTimeout=45s`
   (a thinking VLM's latency is higher+variable, measured 9–15s; 20s caused spurious
   PROVIDER_TIMEOUT + retry churn).

⚠️ **RE-TESTING GOTCHA:** to re-run OCR on a claim you must reset `attempt_count=0` too, not just
`job_status='pending'`+`error_code=NULL`. `ListPendingJobs` filters `attempt_count < maxAutomaticAttempts`
(=3, migration 006 CHECK). After 3 retries the row stays `pending` forever but is never selected
(looks stuck; worker silent because `ProcessPointClaimOCR` only logs on error).

⚠️ **ENV GOTCHA (hand-started services):** backoffice-api main.go has **no godotenv** — it reads
config from the *process* environment. air rebuilds the binary on file change but the child inherits
the air *parent's* env, fixed at launch. A `nohup air` started without `set -a; . ./.env` runs OCR on
`envDefault` (openrouter base + empty key), so provider is misconfigured with NO error until a job
runs. Fix = restart air with `.env` sourced; confirm boot line `Receipt OCR enabled provider=vlm`.
Verified via `ps eww <pid> | grep RECEIPT_OCR` (0 matches = stale env).

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
