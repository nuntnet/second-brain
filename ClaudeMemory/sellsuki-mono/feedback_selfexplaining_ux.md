---
name: feedback-selfexplaining-ux
description: ทุกหน้า/ฟีเจอร์ที่สร้างให้ user ต้องอธิบายตัวเอง — บอกว่ามีไว้ทำอะไร + กดอ่านต่อได้ + บอกผลของ action ก่อนกด
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 06c2449b-af9f-42b9-bf19-e32fb331ac0f
---

จาก review Control Tower (2026-07-18) — user ตี 7 ข้อ ที่สรุปเป็น pattern เดียว: **หน้าที่ข้อมูลแน่นแต่ไม่สื่อสาร = ใช้ไม่ได้**. ก่อนหน้านี้ก็เคยสั่ง "อ่านไม่ออกเลย ทำให้ simple กว่านี้มากๆ" (Thai-first pass).

**Why:** user เอาเครื่องมือไปส่งต่อให้คนใหม่ (น้อง PO) เสมอ — ถ้าหน้าไหนต้องมีคนอธิบาย หน้านั้นล้มเหลว. และ user จะถามเสมอว่า "หน้านี้ทำเพื่ออะไรกันแน่" ถ้า purpose ไม่ชัดใน 1 ประโยคแรก.

**How to apply:**
1. ทุกหน้า/section เปิดด้วย 1-2 ประโยค: "หน้านี้มีไว้ทำอะไร + ใช้ยังไง + ทำอะไรต่อ" (ภาษาไทยธรรมดา)
2. ข้อมูลสรุปทุกชิ้นต้อง**กดอ่านต่อได้** (drawer/link) — ห้ามเป็น dead-end
3. ปุ่ม action ต้องบอก**ผลที่จะเกิด**ก่อนกด (เช่น "accept ยังไม่แก้อะไร — /ct-apply คือตัวแก้")
4. สอง view ที่โชว์ข้อมูลเรื่องเดียวกัน (roadmap↔chain) ต้อง derive จาก source เดียวกัน — hand-baked ซ้ำ = drift แน่นอน
5. section ที่ตอบไม่ได้ว่า "ต่างจาก section อื่นยังไง" → ยุบ (เคส timeline ซ้ำ chain)

เกี่ยวข้อง: [[feedback-decisive-deep-execution]], [[project-control-tower]]
