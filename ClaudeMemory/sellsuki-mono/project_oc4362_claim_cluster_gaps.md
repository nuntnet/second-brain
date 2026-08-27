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
