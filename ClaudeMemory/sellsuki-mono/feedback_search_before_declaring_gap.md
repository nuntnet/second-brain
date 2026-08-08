---
name: feedback-search-before-declaring-gap
description: "ห้ามประกาศว่า \"ไม่มีข้อมูล/เป็น GAP\" ก่อนค้นให้ครบ — เคยเขียนว่าไม่มี pricing ทั้งที่อยู่ใน docs/ โฟลเดอร์เดียวกัน"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 60159299-7c45-49ce-8985-bacd71095aa0
  modified: 2026-08-08T01:57:20.583Z
---

2026-08-07 เขียน Product KB แล้วใส่ "⛔ GAP: ไม่มีราคาของ product ไหนเลย"
ความจริง: ราคา 4 tier ของ BOLA พร้อม capability key รายฟีเจอร์ อยู่ใน
`docs/plan-capability-quota-map.md` §8 — ห่างจากไฟล์ที่กำลังเขียนแค่โฟลเดอร์เดียว
user จับได้ทันที ("pricing แทบไม่เห็น feature gate ก็ไม่เขียน")

**Why:** การเขียน "GAP" มันง่ายกว่าการไปหา และมันดูเหมือนความซื่อสัตย์
แต่จริง ๆ คือการยกภาระให้ user ไปหาเอง แล้วยังทำให้เอกสารดูว่างเปล่าทั้งที่ของมีอยู่

**How to apply:** ก่อนเขียนว่าอะไร "ไม่มี" ต้อง
1. `ls` + `grep` ทั้ง `docs/` และ `.claude/knowledge/` ด้วยคำที่เกี่ยวข้อง **หลายคำ** (ไทย+อังกฤษ+สัญลักษณ์ เช่น ราคา/pricing/฿//เดือน/plan/tier)
2. เช็คชื่อไฟล์ที่บอกใบ้เนื้อหา — `plan-capability-quota-map.md` ประกาศตัวเองอยู่แล้วว่ามีอะไร
3. เช็คใน code ว่ามี entity/enum ที่สื่อถึงเรื่องนั้นไหม
4. เช็ค memory index
เขียน GAP ได้ต่อเมื่อค้นครบแล้วจริง ๆ และควรบอกด้วยว่าค้นที่ไหนมาแล้วบ้าง

ญาติกับ [[feedback-ground-claims-file-line]] และ [[feedback-verify-as-the-user-sees-it]]
