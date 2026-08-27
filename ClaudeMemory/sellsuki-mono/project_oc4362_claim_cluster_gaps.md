---
name: project_oc4362_claim_cluster_gaps
description: OC-4362 point-claim cluster — การ์ดใหม่ OC-4461..4465 อยู่ Sprint 128 + naming/schema blockers ที่ยังค้าง
metadata: 
  node_type: memory
  type: project
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-08-26T04:39:22.603Z
---

2026-08-26: เปิดการ์ดต่อยอด OC-4362 (ใบเสร็จไม่มี QR) / OC-4407 (marketplace) เข้า epic OC-2743:
**OC-4461** review-queue shell (เมนู "คำขอแต้ม" 2 แท็บ + permission) · **OC-4462** member "คำขอของฉัน" ·
**OC-4463** LINE push แจ้งผลตรวจ · **OC-4464** OCR-assist (human-approve เสมอ) ·
**OC-4465** guest-first claim (ส่งใบเสร็จก่อน สมัครทีหลัง — draft ใน IndexedDB + return_to allowlist)

2026-08-27: ย้ายทั้งคลัสเตอร์ 9 ใบเข้า **OC Sprint 128 = id 1364** (board 2, 6–20 ก.ย. 2026):
OC-4295/4362/4413/4420/4461/4462/4463/4464/4465 — ก่อนหน้านี้ 4362/4413/4420 ไม่มี sprint เลย

**Blocker ที่ยังค้าง (ยังไม่มีใบไหนผ่าน DoR ตอนย้ายเข้า sprint):**
- schema ของ claim ที่ OC-4362 ยังไม่ freeze แต่ 4462/4463/4464 ผูก field ไว้หมด
- ชื่อ scope ชนกัน: `receipt-claim-review` (4362/4407) vs `oc2plus.pointclaim.review` (4461 — format จริงของ platform, constant gen ในโมดูล `entity`)
- `claim_source: receipt|marketplace` (4464) vs canonical `channel: manual|marketplace` (OC-4413 contract 43344)
- OC-4465 ยัง blocked-by OC-4345 (auth spike) — ต้องเคาะ D1 (ฟอร์มอยู่ SPA ไหน) + D2 (guest→member upgrade ในเซสชันเดียว) ไม่งั้นทำไม่ได้ทั้งใบ
- link ทิศกลับด้าน 3 จุด (MCP ลบไม่ได้ ต้องแก้ใน Jira UI): OC-4362 link 23390, OC-4407 link 23380, และ OC-4362 ขาด is-blocked-by OC-4420

**ผลตรวจ codebase (แก้แล้ว 2026-08-27 — รอบแรกผมอ่านผิด base):**
🔴 local `develop` ของ backoffice-api **แตกทางจาก origin/develop: ตามหลัง 137 / นำหน้า 191 commit**
→ สำรวจจาก local develop จะได้ข้อสรุปผิด **ยึด `origin/develop` เสมอ**

ของจริงบน origin/develop:
- เก็บไฟล์ผ่าน **file-service ทาง HTTP** (`src/repository/file_service_repository/http.go:66`) — ตรงกับที่การ์ดเขียนไว้แล้ว
- **ไม่มี** `use_case/file_storage.go` ⇒ เรื่อง S3-ตรง / `ExpiresAt` 24 ชม. / ตาราง `file_storage` **เป็นของ local ที่ stale ทั้งหมด ไม่ใช่ blocker**
- 🔴 blocker จริง: มีแค่ `UploadPublicImage` → `/upload/public/` (ทำไว้ให้ theme logo OC-4246)
  **ยังไม่มี private upload** — รูปใบเสร็จเป็นข้อมูลลูกค้า ใช้ public bucket = ปัญหา PDPA
- `member-api` ยังไม่มี point/loyalty/claim endpoint และไม่มีโค้ดอัปโหลดไฟล์

ดู [[reference_oc2plus_jira_project]], [[project_loyalty_canonical_contract]], [[project_loyalty_point_cluster]]

---

## 2026-08-28 — สถานะหลัง implement (MR เปิดครบ ชี้ `develop` ทั้งหมด)

| การ์ด | สถานะจริง | MR |
|---|---|---|
| OC-4362 | ฝั่ง member ครบ (submit + ดูรูปตัวเอง) · admin proxy + reject ครบ · **approve ยังไม่ทำ** (ติด OC-4415) | member-api !93 · backoffice-api !521 · FE !551 / !35 |
| OC-4462 | ครบ | FE member !35 |
| OC-4463 | ครบ | backoffice-api !521 |
| OC-4461 | **code ครบทุก AC** — scope เป็น `oc2plus.pointclaim.review` แล้ว (entity v0.32.0) | รอ **ops grant scope** ก่อน deploy · BE+FE ต้องขึ้นพร้อมกัน |
| OC-4465 | BE idempotency + FE guest/modal/draft ครบ · **AC-02 pre-check ถูก descope** (member-api ไม่มี campaign repository — เหตุผลเดียวกับที่ OC-4362 ตัด Rule 4) | !93 · !35 |
| OC-4464 | ยังไม่เริ่ม — ต้องเคาะ OCR vendor ก่อน | — |

**D1/D2 ของ OC-4465 ตอบแล้วโดยของที่ ship** ไม่ต้องรอ spike OC-4345: ฟอร์มอยู่ SPA เดียวกับ register
(`frontend/oc2plus-linecrm-frontend-member`) และ guest = ไม่มี cookie เลย ไม่มี bootstrap session บังคับ

**Deploy blocker ที่ยังไม่ได้ทำ (ต้องให้ SRE):** grant `sellsuki.filesystem.{create,view,list}` ให้
identity ของ `FILE_SERVICE_API_KEY` **kind `sellsuki.system`** ทุก company — ไม่มีอะไรใน CCS/rps แจกให้ใครเลย
ดู [[reference-file-service-keto-subject-kind]]

**ยังไม่เคย verify ใน browser จริงสักหน้า** — backoffice ติด Kratos ปฏิเสธ return_to=localhost (ต้องมีคน login),
member LIFF บูตไม่ขึ้นเพราะ dep หาย ดู [[reference-browser-surfaces-this-workspace]]

**2026-08-28 — OC-4461 ปิด code ครบ**: entity!44 merged → tag `v0.32.0` (commit `c0124d1`) →
backoffice-api go.mod v0.28.0→v0.32.0 + สลับ 6 จุดใน `point_claim.go` → FE เปลี่ยน sidebar + page guard
เป็น `PERMISSIONS.POINT_CLAIM_REVIEW`

`src/use_case/point.go` และ `ViewPointList` **คงเป็น `oc2plus.point.manage`** — คนละฟีเจอร์ (จัดการหน่วยแต้ม)
`App.vue` route-permission map คุมแค่ panel export ไม่ใช่ page access จึงไม่ต้องเพิ่ม `point-claims`

⚠️ **ห้าม deploy ก่อน ops grant scope** — เมนูจะหายจากแอดมินทุกคน (ถูกตาม Rule 9 แต่ใช้งานไม่ได้)
และ **ห้ามถอยกลับไปใช้ point.manage** เพราะนั่นคือ bypass ที่ Rule 9 มีไว้กัน

⚠️ **push tag ถูก classifier บล็อก** — สร้าง local tag ได้ push ไม่ได้ ต้องให้ user ทำเอง
ดู [[reference-harness-classifier-blocks-secrets-and-mutations]]

OCR vendor ของ OC-4464 เคาะแล้ว ดู [[project-oc4464-ocr-vendor-decision]]

