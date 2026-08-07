---
name: project-sellsuki-product-kb
description: Product KB (BD/MKT/Sales/C-level) อยู่ที่ docs/product-kb/ — ป้ายกำกับ verified/asserted/hypothesis/GAP บังคับทุก claim
metadata: 
  node_type: memory
  type: project
  originSessionId: 60159299-7c45-49ce-8985-bacd71095aa0
  modified: 2026-08-07T16:14:42.098Z
---

Product Knowledge Base สำหรับ **BD · Marketing · Sales · C-level** อยู่ที่ `docs/product-kb/`
(เขียน 2026-08-07 ตามโครง `~/Downloads/Sellsuki-Product-KB-Outline.md`) — 11 ไฟล์:
README (สารบัญ+กติกา) · 00 portfolio overview · 01 AI chatbot capability ·
02-07 Shipmunk/OC2Plus/SukiPay/Patona/BOLA/Akita · `_gaps.md` · `_template.md`

**กติกาที่ต้องรักษาไว้:** ทุก claim ติดป้าย ✅ verified (มี file:line / Jira / data.js) ·
🟡 asserted · 🔬 hypothesis · ⛔ GAP — เพราะเอกสารนี้จะถูกเอาไปพูดกับลูกค้าจริง
pain point ต้องแยก evidence จริง (Jira bug) ออกจากสมมติฐานเสมอ
(องค์กรยังไม่มี user research จริง ดู [[project-user-pain-evidence-gap]])

**ข้อสรุปที่ได้จากการเขียนรอบนี้ (อย่าเสียเวลาค้นซ้ำ):**
- AI Chatbot **ไม่ใช่ engine เดียว** — 3 ระบบแยกกัน: `rag-core`+channel-gateway (Python/Milvus,
  live) · BOLA KB (Go, OpenAI embeddings จริง ไม่มี ANN index) · AI Chat Platform (กำลังสร้าง)
- Shipmunk **มี public API + API key auth อยู่แล้ว** — roadmap P2 ที่บอกว่าต้องสร้างนั้นผิด
  gap จริงคือ self-service onboarding + billing + white-label
- Shipmunk มี carrier adapter 6 เจ้าใน code (dhl/flash/jnt/kerry/ninjavan/thaipost) แต่ README บอกแค่ 2
- ช่องที่เป็นหลุมใหญ่สุด = **HOW** (pricing / SLA / ToS / support ไม่มีเลยทุก product) → `_gaps.md` G-02/G-10/G-11/G-12

**Surface:** Control Tower แท็บ Docs มีชั้นวาง "📕 Product Knowledge Base" ปักหมุด (แก้ที่ `index.html` `#kbShelf` + `KB_SHELF`)
**Outline:** ยังไม่ได้ publish — ติด VPN ดู [[reference-outline-mcp-vpn-blocker]]

commit `c850f79` บน branch `feat/oc-4200-member-follower` (monorepo local-only ดู [[reference-monorepo-no-origin]])
