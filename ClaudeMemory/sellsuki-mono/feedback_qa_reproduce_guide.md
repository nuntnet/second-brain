---
name: feedback_qa_reproduce_guide
description: "การ์ดต้องมี section \"QA — Reproduce & Test Guide\" — ขั้น reproduce บั๊กปัจจุบันแบบกดตามได้ + ตารางเทส map AC + test data ที่ต้องเตรียม"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
---

User feedback (2026-07-10, ชุดการ์ด invite chain PAT-2553/2554/OC-4371): "การ์ดที่เขียนทั้งหมด อ่านแล้วงง ว่า qa จะเทส reproduce flow ที่ขาดอย่างไรบ้าง"

**Why:** การ์ดที่บอกแค่ "อะไรพัง + AC" ไม่พอ — QA ต้องเดินตามได้โดยไม่ตีความเอง

**How to apply:** ทุกการ์ด (โดยเฉพาะ bug/gap-driven) ใส่ section `QA — Reproduce & Test Guide` ประกอบด้วย (1) เตรียมของก่อนเทส: users/companies/invites/เครื่องมือที่ต้องมี (2) ขั้นตอน reproduce อาการปัจจุบันก่อนแก้ ระบุ "จุดสังเกต" ที่เห็นบั๊ก เช่น ดู address bar/Network tab (3) ตารางเทสหลังแก้ที่ map ทุกแถวเข้า AC — ชุดเดียวพิสูจน์ได้ทั้ง "บั๊กมีจริง" และ "แก้แล้วหายจริง" — เกี่ยวกับ [[feedback_card_user_story_flows]]
