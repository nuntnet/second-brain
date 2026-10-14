---
name: project-mahanakhon-proposal-vs-roadmap-decisions
description: "2026-10-14 ผู้ใช้เคาะการตัดสินใจที่เบี่ยงจาก Proposal V.1 มหานครโลหะ (CRM เป็นเจ้าของ active/inactive + tier, แยกตาม company, เดือนเกิด/manual/QR ลูกค้าขอในที่ประชุม) · Proposal เป็น PDF ใน Downloads เปิดผ่าน Drive ไม่ได้"
metadata:
  node_type: memory
  type: project
  originSessionId: 846eb860-82e8-4469-8af8-09ce62f37397
  modified: 2026-10-14T13:36:06.064Z
---

เทียบ roadmap (oc2plus-mahanakhon-roadmap.md) กับ Proposal V.1 "มหานครโลหะ CRM & LINE API" (24 หน้า, Sellsuki × WizeMoves) แล้ว ผู้ใช้ตอบจุดที่ขัดกันไว้ดังนี้ (2026-10-14):

- **Active/Inactive และ Tier เป็นของ Oc2plus (CRM)** ไม่ใช่ Scrapyard ตาม Proposal. สมัคร LINE โดยไม่มี contact ก่อน = เริ่ม inactive; active เมื่อเคยซื้อขาย. Scrapyard ต้อง **รับ webhook** สถานะ/tier แล้วเปลี่ยนกลุ่มราคารับซื้อเอง
- **แยกลูกค้าตาม company เท่านั้น** (1 สาขา = 1 company) ไม่ทำ branch_code logic ใน CRM — ตรงกับ Proposal ที่เขียน "สร้าง Company หลาย Company (สาขา)"
- เดือนเกิด, สถานะ manual, สมัคร LINE ก่อน, QR ท้ายใบเสร็จ, แต้มบนใบเสร็จ, PDF = **ลูกค้าขอเพิ่มในที่ประชุม** ไม่ใช่ความผิดที่ไม่อยู่ใน Proposal
- ยอดสะสมแยกหมวด/กก. + SKU campaign ถูกเลื่อน P3 **เพราะติด blocker การ์ดอื่น** (ไม่ใช่ลูกค้ายอม)
- มินิเกม/ข่าว/banner/ช่องทางขาย → ทำเป็น Custom CMS microfrontend ให้ทีม WizeMoves martech ทำหน้า
- BOLA ทำตาม spec Proposal ทั้งหมด
- Webhook ต้องแจ้งเมื่อ create/update member info, tier, status, point

**Why:** Proposal V.1 เขียนตรงข้ามกับการตัดสินใจเหล่านี้ — อย่าเรียกว่า "ผิด" และอย่าเสนอย้ายกลับตาม Proposal. แต่ฝั่ง Scrapyard งานเปลี่ยน (ส่ง→รับ webhook) และค่า API ฝั่งเขาไม่อยู่ใน Proposal (T&C ข้อ 3) จึงควรมีหลักฐานเป็นลายลักษณ์อักษร

**How to apply:** เทียบ roadmap/การ์ดกับ Proposal ให้ถือการตัดสินใจข้างบนเป็น source of truth. ช่องว่างที่เจอและยังไม่ปิด: OC-4428 ไม่มี webhook event เปลี่ยน tier / member.updated; OC-2261 (เปลี่ยนเบอร์ re-OTP, Ready to test DEV) ไม่อยู่ใน roadmap; รายงาน 8 ชุด/ปัดเศษ-หมดอายุแต้ม/แลกของรางวัล+ที่อยู่ ต้อง reconcile กับโค้ดและ Jira; แผนภาพ Proposal ให้ BOLA ดู follower 14 สาขาในที่เดียว (ข้าม company). ไฟล์: ~/Downloads/Proposal - มหานครโลหะ CRM & LINE API_V.1.pdf (Google Doc ลิงก์เปิดผ่าน Drive MCP ไม่ได้ — คนละบัญชี). เกี่ยวข้อง [[reference-oc2plus-jira-project]]
