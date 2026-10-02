---
name: project-sellsuki-product-kb
description: Product KB (BD/MKT/Sales/C-level) อยู่ที่ docs/product-kb/ — ป้ายกำกับ verified/asserted/hypothesis/GAP บังคับทุก claim
metadata: 
  node_type: memory
  type: project
  originSessionId: 60159299-7c45-49ce-8985-bacd71095aa0
  modified: 2026-08-07T16:43:27.943Z
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
- **ราคา BOLA มีจริง** ใน `docs/plan-capability-quota-map.md` §8 (Starter ฿590 / Pro ฿1,990 / Ent ฿4,990 + capability key รายฟีเจอร์ + เพดาน workspace/broadcast/ai_message) — §7 = กติกา capability (boolean by presence, ต่อ feature ไม่ใช่ต่อ product)
- **การล็อกสิทธิ์ตาม tier ยังไม่บังคับใช้** ทุก workspace = enterprise ตาม migration → sale ห้ามใช้ข้อจำกัดฟีเจอร์ปิดการขาย
- **OC2Plus ไม่มีราคาเลยจริง ๆ** (ค้นครบแล้ว 2026-10-02: แผนราคากลาง + `docs/` + การ์ด OC ทุกใบ)
  → ใส่เป็น **ตารางเปล่า** ไว้แทน แยกเพดานปริมาณ 12 แถว ออกจากสิทธิ์ฟีเจอร์ 16 แถว + คีย์ที่เสนอ + 7 คำถามที่ต้องตอบ
  และ **OC2Plus ไม่บังคับใช้แพ็กเกจเลย** ดู [[project-oc2plus-no-packaging-enforcement]]
- Patona/SukiPay/Shipmunk ยังไม่ได้ขุด pricing

**Surface:** Control Tower แท็บ Docs มีชั้นวาง "📕 Product Knowledge Base" ปักหมุด (แก้ที่ `index.html` `#kbShelf` + `KB_SHELF`)
**Outline:** publish แล้ว 2026-08-07 → collection **Product Knowledge Base** `9c9911d3-b918-4010-b31b-485551e37e29`
(12 หน้า nest ใต้ README — เพิ่มหน้า 08 เส้นแบ่ง OC2Plus·BOLA·AI) · sync ด้วย `python3 docs/product-kb/publish-to-outline.py --publish` (ต้องอยู่บน VPN
ดู [[reference-outline-mcp-vpn-blocker]]) · **ต้นทางคือ repo เสมอ — ห้ามแก้ใน Outline ตรง ๆ** เพราะ sync รอบหน้าเขียนทับ

commit `c850f79` + `a719b96` บน branch `feat/oc-4200-member-follower` (monorepo local-only ดู [[reference-monorepo-no-origin]])

## รอบ 2026-10-02 — OC2Plus เขียนใหม่ด้วย PMM voice แล้ว publish

เขียนใหม่ทั้งหน้า (ร่าง ส.ค. ล้าไปมาก ตอนนั้นมี backend ตัวเดียว ตอนนี้มี 3 + point claim/OCR +
tier + coupon wallet + API key v2 + MCP) · BOLA กับ OC2Plus = **2 หน้าที่ผ่านมาตรฐานใหม่แล้ว**
เหลือ Patona · SukiPay · Shipmunk · Akita ที่ยังเป็นสไตล์ auditor เดิม

**คำถาม positioning ที่ user ยังไม่เคาะ:** tagline ทางการใน §0.4 เขียนว่า OC2Plus =
"CRM + CDP + Messaging" แต่หน้าใหม่ไม่ได้ขาย Messaging เลย (กล่องแชทรวมยังไม่มี · หน้า 08 ระบุว่า
ข้อความการตลาดมีที่เดียวคือ BOLA) → **tagline สัญญาเกินของที่มี** ยังไม่แก้เพราะเป็นการตัดสินใจของ user
