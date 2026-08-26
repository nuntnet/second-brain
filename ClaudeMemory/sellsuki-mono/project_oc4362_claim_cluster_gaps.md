---
name: project_oc4362_claim_cluster_gaps
description: OC-4362 point-claim cluster — 4 การ์ดใหม่ OC-4461..4464 + naming/schema blockers ที่ยังค้าง
metadata: 
  node_type: memory
  type: project
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-08-26T04:39:22.603Z
---

2026-08-26: เปิดการ์ดต่อยอด OC-4362 (ใบเสร็จไม่มี QR) / OC-4407 (marketplace) เข้า epic OC-2743:
**OC-4461** review-queue shell (เมนู "คำขอแต้ม" 2 แท็บ + permission) · **OC-4462** member "คำขอของฉัน" ·
**OC-4463** LINE push แจ้งผลตรวจ · **OC-4464** OCR-assist (human-approve เสมอ)

**Blocker ที่ยังค้าง (0/4 ผ่าน DoR):**
- schema ของ claim ที่ OC-4362 ยังไม่ freeze แต่ 4462/4463/4464 ผูก field ไว้หมด
- ชื่อ scope ชนกัน: `receipt-claim-review` (4362/4407) vs `oc2plus.pointclaim.review` (4461 — format จริงของ platform, constant gen ในโมดูล `entity`)
- `claim_source: receipt|marketplace` (4464) vs canonical `channel: manual|marketplace` (OC-4413 contract 43344)
- link ทิศกลับด้าน 3 จุด (MCP ลบไม่ได้ ต้องแก้ใน Jira UI): OC-4362 link 23390, OC-4407 link 23380, และ OC-4362 ขาด is-blocked-by OC-4420

**ผลตรวจ codebase ที่ขัดกับ OC-4362:**
- การ์ดเขียน "อัปโหลดผ่าน file-service" แต่ของจริงคือ **S3 ตรง** (`s3.go:38`, `use_case/file_storage.go:17`)
- ไฟล์แนบตั้ง `ExpiresAt` **+24 ชม.** (`use_case/file_storage.go:36`) → หลักฐานอาจหายก่อนแอดมินตรวจ
- `oc2plus-line-crm-service-member-api` มีแค่ **9 endpoints** ไม่มี point/loyalty/claim และ **ไม่มี code อัปโหลดไฟล์เลย** → ฟอร์มฝั่ง member ยังไม่มีบ้าน

ดู [[reference_oc2plus_jira_project]], [[project_loyalty_canonical_contract]], [[project_loyalty_point_cluster]]
