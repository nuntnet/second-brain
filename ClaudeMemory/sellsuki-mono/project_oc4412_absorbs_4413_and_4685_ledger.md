---
name: project-oc4412-absorbs-4413-and-4685-ledger
description: "2026-10-01 OC-4412 รวม OC-4413 (engine, AC-E) และส่วนเขียน ledger ยอดขายของ OC-4685 (AC-L) — RecordSalesActivity เรียกจาก Commit ให้ครอบทุกช่องทาง · OC-4685 เหลือค่าสรุป+การ์ด+BOLA sync"
metadata:
  node_type: memory
  type: project
  originSessionId: 53f9c3a5-efcb-4605-8af2-6e152327b40b
  modified: 2026-10-01T15:32:31.034Z
---

วันที่ 2026-10-01 PO ตกลงสองเรื่อง

1. **OC-4413 รวมเข้า OC-4412**
   - AC ของ 4413 กลายเป็น **AC-E01..E21**
   - 4413 ปิดแล้ว
2. **ส่วนเขียนของ OC-4685 ย้ายเข้า OC-4412**
   - ที่ย้าย: ตาราง `member_sales_activity` / `_measure` / `member_sales_monthly`, `RecordSalesActivity`, `measures[]` และ void
   - AC ที่ย้ายกลายเป็น **AC-L01..L09** และมี AC ใหม่ L02b กับ L10
   - เลขเดิมยังเทียบได้: `OC-4685 AC-0x` = `OC-4412 AC-L0x`
   - OC-4685 เหลือแค่ฝั่งอ่าน: AC-07b, AC-08r, AC-10/10b และ AC-11..18 (BOLA)
   - OC-4685 ถูก block โดย OC-4412

**การออกแบบ:**
- `RecordSalesActivity` ถูกเรียกจาก `CommitAward` ไม่ใช่จาก HTTP adapter
  - ผลคือทุก caller ของ engine ได้ ledger ทั้ง POS sync, Kafka OC-4414 และการอนุมัติคำขอแต้ม (OC-4575 ผ่าน OC-4572)
- plan ว่าง หรือสมาชิก `pending_claim` → เขียน ledger ใน txn ของตัวเอง
  - AC-C08 ไม่เปลี่ยน: plan ว่างไม่เขียน dedup row
- migration อยู่รีโป 530 พร้อมสำเนาใน member-api ([[reference-oc2plus-schema-lives-in-external-repo]])

**Why:** ledger ต้องอยู่ใน txn เดียวกับ Commit และผู้เขียนมีจุดเดียว ถ้าแยกใบจะได้สอง MR บนโค้ดก้อนเดียวกัน

**How to apply:**
- งานยอดขายต่อใบเสร็จให้ไปดูที่ OC-4412 ไม่ใช่ 4685
- ณ 2026-10-01 โค้ด ledger ยังไม่เริ่ม (branch `feat/oc-4412-purchase-award` มีแค่ adapter กับตัวโหลด config)
- เรื่องที่ยังค้าง PO เคาะ:
  - OQ-9: อัตราพื้นฐานใช้กับหน่วยหลักหรือทุกหน่วย
  - OQ-17: ใบที่ได้ 0 แต้ม ถ้าส่งซ้ำทีหลังจะได้แต้มย้อนหลังหรือไม่
  - OQ-19: ไม่มี cache
  - OQ-20: บิลจากการอนุมัติคำขอแต้มไม่เข้า ledger จนกว่า OC-4575 จะเรียก Commit ตัวนี้

ดู [[project-award-engine-has-no-callers]] และ [[project-oc4469-oc4428-card-merge]]
