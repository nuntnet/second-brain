---
name: feedback-product-marketing-voice
description: "เขียนเอกสาร product/marketing ต้องสวมบท PMM ไม่ใช่ auditor — pain=งานลูกค้าไม่ใช่ bug, ต้องมีสูตรผสมฟีเจอร์, ToS/SLA ให้ร่างไม่ใช่ประกาศ GAP"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 60159299-7c45-49ce-8985-bacd71095aa0
  modified: 2026-08-08T01:56:53.419Z
---

ตอนเขียน Product KB รอบแรก (2026-08-07) user ตีกลับว่า "ใช้อธิบาย/ทำการตลาดแทบไม่ได้เลย"
รอบสองผ่าน — ความต่างอยู่ที่ **บทบาทที่สวม**

**Why:** เอกสารพวกนี้ให้ BD/MKT/Sales/C-level ใช้ปิดดีล ไม่ใช่ให้ engineer ตรวจสอบความจริง
สวมบท auditor เมื่อไหร่ ผลลัพธ์จะเป็นตารางป้ายกำกับ + สถานะ Jira ที่ไม่มีใครเอาไปใช้ได้

**How to apply — บทที่ต้องสวม: Product Marketing Manager ที่เคยเป็น solution consultant**
เกณฑ์ตัดสิน: *sale อ่านหน้าเดียวแล้วเข้าห้องประชุมได้เลยไหม*

1. **Pain point = งานที่ลูกค้าพยายามทำแล้วติดเพดานเครื่องมือเดิม — ไม่ใช่ bug ของเรา**
   ห้ามเอา Jira bug / QA finding มาใส่ช่อง pain point (พลาดรอบแรก) · bug = "ของเราพัง" คนละเรื่องกับ "ก่อนมีเราลูกค้าเจ็บตรงไหน"
2. **Core value ≠ list ฟีเจอร์** — ต้องเป็นสมการธุรกิจ เช่น BOLA = "ทุกบาทค่าข้อความ LINE ไปถึงคนที่มีโอกาสซื้อ" ไม่ใช่ "มี broadcast+segment+AI+rich menu"
3. **ต้องมีสูตรผสมฟีเจอร์ (use case จริง)** — user เน้นข้อนี้มากที่สุด "หลายครั้งมันใช้ผสมกัน เช่น autoreply แยกตาม segment ได้"
   → ก่อนเขียนต้องไปอ่าน code ว่าอะไรผสมกับอะไรได้จริง (เช่น BOLA: segment เป็นตัวเชื่อม autoreply/APM/broadcast/rich menu)
4. **ความเจ๋งที่ user มองว่าเจ๋ง = craft ที่กันบั๊กทั้งชั้น และโผล่เป็นความน่าเชื่อถือที่ลูกค้าสัมผัสได้**
   เช่น บอกผลก่อนกด (segment preview) · กันส่งซ้ำ · fail-closed · auto-provision ลดขั้นตอนที่ลูกค้าพลาด
   ผูกกับ [[feedback-selfexplaining-ux]]
5. **ToS / SLA / Customer Support / Release plan → ให้ "ร่าง" มา ไม่ใช่เขียน ⛔ GAP**
   ร่างจากความสามารถจริงของระบบ แล้วติดป้ายว่ารอ legal/management อนุมัติ
   SLA ต้องแยก "สิ่งที่เรารับประกันได้" ออกจาก "สิ่งที่ขึ้นกับ platform ภายนอก (LINE)" และแยก SaaS vs self-hosted
6. **อย่าโรยป้าย ✅🟡⛔ ทั้งเอกสาร** — รวบเป็นบล็อกท้ายหน้า "พูดได้ / ยังพูดไม่ได้" แทน ความซื่อสัตย์ยังอยู่ครบแต่ไม่ขวางการอ่าน

ใช้กับทุก product ใน [[project-sellsuki-product-kb]] · ญาติกับ [[feedback-card-user-story-flows]] (การ์ดห้ามเป็น technical concept ล้วน)
