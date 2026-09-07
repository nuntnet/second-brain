---
name: feedback_oc2plus_merge_to_develop
description: OC2Plus MRs ทุกตัวต้อง merge to develop เท่านั้น ห้าม merge to main
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-09-07T14:13:08.207Z
---

**กฎ (user สั่ง 2026-07-20):** MR ของงาน OC2Plus **ทุกตัวต้อง target `develop` เท่านั้น ห้าม merge เข้า main**

**ครอบคลุมถึง platform service กลางด้วย** (group `sellsuki/sellsuki`) เมื่อ MR นั้นเป็นส่วนของ OC2Plus delivery — user ยืนยัน "ครอบทั้งหมด → develop" รวม role-permission, CCS, ไม่ใช่แค่ repo ที่ OC2Plus เป็นเจ้าของ

**How to apply:** ตอนสร้าง/ตรวจ MR ของงาน OC (เช่น OC-4257 QMS chain) — เช็ค target branch เป็น develop; ถ้าเผลอ target main ให้ `glab mr update <id> --target-branch develop` ทุก repo มี `origin/develop`

**Why:** main = release branch, develop = integration; OC2Plus flow รวมงานที่ develop ก่อน

QMS delivery 2026-07-20: !194 (mgmt-backend), !7 (provider-fe), !86 (role-permission), !253 (CCS feature/provider) — retarget → develop หมด · !194/!7 สะอาด, !86/!253 ต้องแก้ conflict (develop นำหน้า) · feature/provider แยก QMS commit ออกไม่ได้ (พึ่ง checkPermissionForCompanyOperation จาก PAT-2448 03b000f) เชื่อม [[project_qms_ui]]

⚠️ **AUTO-MR-TO-MAIN TRAP (backoffice-api, 2026-09-07):** a plain `git push origin <new-branch>`
on `oc2plus-line-crm-service-backoffice-api` **auto-created an MR targeting `main`** (repo default),
even with `-o merge_request.create=false`, and stamped it with an **unrelated default-template title**
(showed "feat(BOLA-317)…" over my OC-4362 commits). Had to `glab mr close`. When pushing a NEW branch
here for preservation, immediately check `glab mr list --source-branch <b>` and close/retarget any MR
that lands on main. Prefer opening the real MR yourself with `glab mr create --target-branch develop`.

**OC2Plus loyalty landing state 2026-09-07** (all MRs → develop): member-api **!98**
(feat/oc-4407-marketplace-claim — now carries OC-4407 + OC-4348 OTP + §C image-hash + §B read_items,
pushed 2631133..712285d); member-FE **!35** (feat/oc-4462-my-claims — + DUPLICATE_IMAGE + success-fix
+ OTP page, 3f1b36b..75cc83c); backoffice-FE **!572** NEW (feat/oc4362-approve-button — admin approve/
edit/OCR-items UI, blocked on backend). backoffice-api backend (§B/§A/§C-4420/OCR-items/vlm-fix,
6 commits on `feat/oc-4362-admin-edit-claim`, pushed for preservation, backup branch
`backup/oc-4362-admin-edit-local`) is **GATED on approve MR !538** — no clean base exists until !538
lands (develop already has OCR providers 4383696; approve foundation 24921e4/f0016f2 is only in !538,
and !538's branch predates the OCR providers so rebasing onto it modify/deletes vlm.go). Post-!538
landing = `git rebase --onto origin/develop 9da44df feat/oc-4362-admin-edit-claim` then
`glab mr create --target-branch develop`. See [[project_oc4362_approve_and_admin_edit]].
