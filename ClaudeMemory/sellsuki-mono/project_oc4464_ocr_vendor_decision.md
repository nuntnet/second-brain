---
name: project-oc4464-ocr-vendor-decision
description: "OC-4464 OCR vendor: เคาะ iApp ไว้ 2026-08-28 แต่ 2026-09-10 เลิกใช้แล้ว ของจริงคือ VLM (OpenRouter gemini-2.5-flash); adapter iApp ถูกลบ, default = vlm — และ Textract ใช้ไม่ได้เพราะไม่มีภาษาไทย"
metadata:
  type: project
---

2026-08-28 — เคาะ vendor สำหรับ OCR-assist ใบเสร็จ (OC-4464): **iApp Thai Receipt OCR**
(https://iapp.co.th/docs/thai-document-optical-character-recognition/receipt)

**เหตุผล:** คืน *field ของใบเสร็จไทย* ไม่ใช่ *ข้อความไทย* ที่ต้อง parse เอง — ใบเสร็จไทยไม่มีฟอร์แมต
มาตรฐาน การเขียน regex ไล่หาว่าอันไหนเลขใบเสร็จ/ยอด/วันที่ คือจุดที่โปรเจกต์แบบนี้ตาย · เป็นบริษัทไทย
→ PDPA เป็น DPA ในประเทศ ไม่ใช่ cross-border ซึ่งสำคัญเพราะรูปใบเสร็จอยู่ใน private bucket โดยตั้งใจ

**🔴 AWS Textract ใช้กับใบเสร็จไทยไม่ได้** — รองรับแค่ EN/ES/IT/PT/FR/DE (forms/tables แคบกว่านั้นอีก)
ไม่มีไทย ยืนยันจาก AWS FAQ 2026-08 · ถ้ามีใครเสนอมาให้ปัดทิ้งได้ทันที ไม่ต้องเสียเวลา PoC

**สำรอง (ถ้ารูปออกนอกระบบไม่ได้):** Typhoon OCR ของ SCB 10X — open weights รันเอง ไม่มี third party
เคลมชนะ GPT-5 / Gemini 2.5 Flash บนเอกสารไทย layout ซับซ้อน · org มี `rag-core` อยู่แล้ว

**Azure Document Intelligence:** Read/Layout รองรับไทย แต่ **ยืนยันไม่ได้ว่า prebuilt receipt model
รองรับไทย** — docs เคลม 100+ ภาษาแต่ตาราง locale ไม่ยืนยัน ต้องเช็คเองก่อนพิจารณา

**กับดัก พ.ศ. (สำคัญที่สุด):** ใบเสร็จไทยพิมพ์ปี 2569 ถ้าเก็บดิบลง `purchase_date` จะได้วันที่ล่วงหน้า
543 ปี → validation ของ OC-4362 (ไม่เกิน 30 วันย้อนหลัง, ไม่อยู่ในอนาคต) reject ทุกใบแบบเงียบ
ต้องเป็นเคสทดสอบแรกเสมอ

วัดผลเป็น "กี่ใบที่แอดมินไม่ต้องแก้เลย" ไม่ใช่ character accuracy — 98% char accuracy ยังหมายถึง
เลขใบเสร็จผิด 1 ตัวได้

Related: [[project-oc4362-claim-cluster-gaps]]

**2026-09-10 — ยกเลิก iApp:** ของที่รันจริงคือ **VLM reader** (`RECEIPT_OCR_PROVIDER=vlm`) ยิงผ่าน
OpenRouter ไปที่ `google/gemini-2.5-flash` (`src/repository/receipt_ocr_repository/vlm.go`) พิสูจน์กับ
ใบเสร็จจริงแล้วอ่าน quantity/unit_price/line_total ได้ · adapter `iapp.go`/`iapp_test.go` ถูกลบ ค่าคงที่
ที่ใช้ร่วมกันย้ายไป `shared.go` (รวม `init()` ที่ตั้ง logger ให้เทสต์ ไม่งั้น paddle test panic) · default
ของ `RECEIPT_OCR_PROVIDER` เปลี่ยนจาก `iapp` เป็น `vlm` เพราะ default เดิม + key ว่าง = เลือก reader ที่
ไม่มีใครมี credential แล้วอ่านล้มเงียบทุก environment · เหลือ `vlm` กับ `paddle` เท่านั้น ชื่ออื่น = ปิด OCR

**กับดักที่เจอตอน audit (2026-09-10):** `RECEIPT_OCR_*` ไม่เคยถูกใส่ใน `deployment/values-*.yml`
เลยสักไฟล์ → OCR ปิดอยู่ทุก environment ตั้งแต่วันแรกที่ merge โดยไม่มี error ให้เห็น (Rule 3 ทำให้คิว
ทำงานปกติ แค่ตารางเทียบว่าง) เพิ่งเติมทั้ง 3 env พร้อม key จาก secret `oc2plus-crm-secret`
