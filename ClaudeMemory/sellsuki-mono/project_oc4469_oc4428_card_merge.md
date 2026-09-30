---
name: project-oc4469-oc4428-card-merge
description: "2026-09-30 รวม 10 การ์ด 3rd-party API-key เหลือ OC-4469 (สมาชิก+แต้ม) กับ OC-4428 (webhook+log) — ใบที่ปิดแล้วคืออะไร, OC-4419/4414 ตั้งใจแยก, เฟส 1/2 ตาม OC-4683"
metadata:
  node_type: memory
  type: project
  originSessionId: 53f9c3a5-efcb-4605-8af2-6e152327b40b
  modified: 2026-09-30T16:19:06.080Z
---

2026-09-30 ผู้ใช้สั่งรวมการ์ด OC-4468/4469/4470/4471/4475/4429/4427/4428 (+ ถามถึง 4414/4419) ผลคือ:

- **OC-4469 = survivor** ของ 4468 (member.read), 4470 (point.redeem), 4471 (point.adjust), 4475 (ตามรอย key), 4429 (จองชื่อ scope) · AC ใหม่รหัส AC-R/S/T/X/J · AC-B1..B10, R-A..R-F เดิมคงไว้
- **OC-4428 = survivor** ของ 4427 (API request log) · AC ใหม่ AC-L1..L9, AC-D1..D4 · AC-W1..W4 เดิมคงไว้
- ใบที่ถูกรวมปิดเป็น Done + summary ขึ้นต้น `[รวมแล้ว → OC-xxxx]` + comment ชี้ไปใบหลัก
- **OC-4419 (integration guide + OpenAPI doc) กับ OC-4414 (Kafka consumer) ตั้งใจไม่รวม** — 4419 เขียนหลัง contract นิ่ง ถ้ารวมจะวนกับ 4469/4428; ผู้ใช้ถามว่า doc รวมหรือยัง ตอบว่ายัง และเสนอให้เทียบเนื้อ 4419 กับของใหม่ (batch, partner_ref, point.redeem/adjust, webhook 5 event) — ยังไม่ทำ ณ ตอนจบ session
- แต่ละใบแบ่ง **เฟส 1** (จำเป็นต่อ go-live มหานครโลหะ, [[project_mahanakhon_scrapyard_integration]]) กับ **เฟส 2** (ไม่อยู่ใน DoD ของ OC-4683)

**Why:** เลือก survivor เป็นเลขเดิมเพราะ OC-4683, 4412, 4684, 4685, 4687, 4688, 4419, PAT-2612 อ้างเลข/รหัส AC ของสองใบนี้อยู่ · เจอจากโค้ด develop ของ 3rdparty-api ว่า member.read + scope `purchase.submit`/`point.adjust` (reserved) merge แล้วทั้งที่ Jira เป็น To Do, OC-4294 Done แล้ว

**How to apply:** ค้นงานเรื่อง 3rd-party API key ให้ดูที่ 2 ใบนี้ก่อน · **ค้างให้ PO เคาะ:** เฟส 2 อยู่ใบเดียวกันหรือตัดเป็นการ์ดตามหลังถ้า go-live มาก่อน (shipping §1/§7) · ยังไม่ได้แก้ description ของ OC-4683 (แค่ comment) และยังไม่ได้อัปเดต docs/cards/OC-4683.md
