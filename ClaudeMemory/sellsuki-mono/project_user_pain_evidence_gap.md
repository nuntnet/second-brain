---
name: project-user-pain-evidence-gap
description: "องค์กรไม่มี user pain research จริง — \"UX Research Repository\" คือ competitive research; หลักฐาน pain ที่แข็งสุด = Jira bugs จาก QA; pain จริงที่ไม่มี epic = address-form cluster + OC loyalty redemption"
metadata: 
  node_type: memory
  type: project
  originSessionId: b0b4a24f-96f9-41ef-b15d-18e86ad938e1
---

**Finding จาก evidence-first pain research (2026-07-04):** Outline "UX Research Repository" เป็น competitive/technical research (เทียบ Shopify/Shipnity/Bigseller + คุย Tech Lead/CPO) — **ไม่มี merchant interview, support ticket log, churn data, NPS ในองค์กรเลย** → หลักฐาน pain ที่ใช้เขียนการ์ดได้จริงตอนนี้คือ Jira bugs ที่ QA เปิด (Tanakorn/Pan/chaiyakon/Patjukom) ซึ่งแคบกว่า pain หน้างานจริงมาก

**Why:** ทุกครั้งที่เขียน epic/card แล้วอ้าง "user pain" ต้องเช็คว่ามี citation จริงหรือเป็น reverse-justification จาก roadmap — audit พบว่า SukiPay Payment Centralization (theme ที่ carded ครบสุด) ไม่มี raw complaint รองรับเลย ขณะที่บั๊ก payment จริง (PAT-2410 race, 2495 channel ผิด, 1870 e-wallet) ไม่ถูกผูกกับ epic การเงินสักใบ

**How to apply:**
- Pain ที่มี evidence แต่ไม่มี epic: **address-form cluster** (PAT-1456/1457/1501/1502/1503/1524 — 6 บั๊ก component เดียว ควรเป็น 1 epic แก้ต้นตอ) และ **OC loyalty redemption 500** (OC-3637/8/9 — end-customer facing ควรเป็น epic ใหม่ฝั่ง OC)
- สมมติฐานที่ห้ามใช้อ้างจนกว่า validate: add-stock ง่ายสำหรับ SME, 1 location หลาย store (มาจาก CPO ไม่ใช่ merchant)
- ควรผลักดันระบบเก็บ CS ticket + merchant interview เป็นวงจรถาวร
- Full report: `.claude/knowledge/user-pain-research-2026-07-04.md` + `epic-quality-audit-2026-07-04.md`
