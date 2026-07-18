---
name: project-akita-patona-migration-strategy
description: ยุทธศาสตร์หลัก (CEO 2026-07-18) — Patona = rewrite ของ Akita; migrate ลูกค้า Akita → Patona; reuse connectors ไม่ rebuild; SukiPay GTM ก่อน
metadata: 
  node_type: memory
  type: project
  originSessionId: 06c2449b-af9f-42b9-bf19-e32fb331ac0f
---

**ยุทธศาสตร์หลักของบริษัท (CEO review 2026-07-18):**
- **Patona คือ rewrite ของ Akita** — เหตุผลที่สร้าง Patona ตั้งแต่แรก = Akita tech debt เยอะ + bad architectural design. **แผน = migrate ลูกค้าจาก Akita มา Patona.** Akita = sunset path (อย่าลงทุน modernize ใหญ่; "Akita as Standalone Licensing" ขัด strategy — มี suggestion รอ confirm)
- **Omni-Channel Aggregation ≠ rebuild**: reuse/refactor Akita marketplace connectors (พิสูจน์ใน prod แล้ว) เข้า Patona · **ทำสำเร็จทีละ marketplace** (ตัวแรก = ที่ลูกค้า Akita ใช้เยอะสุด — ทีมยืนยัน) — นี่คือคำตอบของ openQuestion "ทำไม rebuild ของที่ Akita มี" ใน evidence dossier
- **BOPIS ถอดออกจาก P0** (ตาม evidence: fail 2 ชั้น + persona แต่งขึ้น) — กลับมาต่อเมื่อมี merchant interview
- **Chain ต้อง decoupled** — spokes เดินคู่ขนานไม่รอ hub (SukiPay พิสูจน์ที่ 36%)
- **SukiPay = GTM ตัวแรก** — ทำไปเยอะสุด แยกขาย standalone ได้ (รอ pricing model จาก Business — ยังไม่มีในองค์กร)
- **เอกสารที่ยังขาด:** แผน migrate ลูกค้า Akita→Patona (ใคร/เมื่อไหร่/criteria) — อยู่ใน missing[] ของ initiative

บันทึกใน SoT แล้ว (commit 5c4d3dc): data.js initiative/chain/doNow · truth.json strategy facts · evidence.json openQuestion ปิดแล้ว · gtm.json SukiPay standalone · 2 suggestions รอ confirm ใน queue (Jira descope PAT-2241/2240 + Akita licensing conflict). เกี่ยวข้อง: [[project-control-tower]]
