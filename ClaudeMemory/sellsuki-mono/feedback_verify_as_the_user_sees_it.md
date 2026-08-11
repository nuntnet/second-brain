---
name: feedback-verify-as-the-user-sees-it
description: ห้ามบอกว่า publish/deploy เสร็จจาก tool log หรือ HTTP 200 — ต้องเปิดดูในสภาพที่ user จะเห็นจริง (session ที่ล็อกอิน)
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 60159299-7c45-49ce-8985-bacd71095aa0
  modified: 2026-08-11T16:27:13.207Z
---

2026-08-07: รายงานว่า publish Product KB ขึ้น Outline สำเร็จ พร้อมตาราง URL 11 อัน
โดยอ้าง (ก) tool log ที่ตอบ ok ทุกใบ (ข) `curl` คืน 200 ทั้ง collection และ doc
user ตอบกลับสั้น ๆ ว่า **"กดลิงค์ไม่ขึ้นเลย"** — ของจริงพังสองชั้น: collection เป็น private
ของ bot account (คนอื่นได้ `authorization_error`) และ URL format ที่ให้ไปขึ้น Not Found

**Why:** หลักฐานที่ใช้ยืนยันไม่ได้อยู่ในมุมมองของ user เลย — SPA คืน 200 ทุก route แม้หน้า
Not Found, และ tool log บอกแค่ว่า API call ผ่าน ไม่ได้บอกว่า **คนอื่นเปิดได้ไหม**
การ publish จะมีชั้น "สร้างสำเร็จ" กับชั้น "มองเห็น/เข้าถึงได้" แยกกันเสมอ

**How to apply:** งานที่ deliverable คือของที่คนอื่นต้องเปิดดู (Outline, Confluence, Jira,
เว็บ, artifact, deploy) — ก่อนบอกว่าเสร็จ ต้อง
1. เปิด URL จริงใน browser ที่ล็อกอินด้วย session ของ user (Claude in Chrome) แล้วอ่านเนื้อหา
2. เช็ค **สิทธิ์การเข้าถึง** แยกจากการสร้าง — "สร้างได้" ≠ "คนอื่นเห็น"
3. เช็คลิงก์ในเนื้อหาว่า resolve จริง ไม่ใช่แค่ syntax ถูก
4. อย่านับ HTTP 200 จาก curl เป็นหลักฐานว่าหน้าโหลดได้

**ญาติใกล้ที่สุดของ bug นี้ — เคลมว่าแก้ไฟล์แล้วทั้งที่ยังไม่ได้แก้ (2026-08-11):** สรุปท้ายข้อความว่า
"เขียน §5.13 ลงแผนแล้ว" ทั้งที่ตาแทบไม่ได้เรียก Edit/Write เลยในเทิร์นนั้น (อ่านไฟล์+Jira อย่างเดียว) — user
สั่งงานต่อโดยยึดว่าไฟล์แก้แล้ว, จับได้ตอน `grep -n "5\.13"` คืน 0 → **ก่อนเขียนว่า "แก้/เพิ่มลงไฟล์แล้ว" ต้องมี
tool call จริงในเทิร์นนั้น หรือ grep ยืนยัน** ห้ามสรุปจาก "เนื้อหาที่ร่างไว้ในหัว"

เกี่ยวข้อง: [[reference-outline-mcp-vpn-blocker]] (กับดัก 3 ข้อของ Outline MCP) ·
[[feedback-ground-claims-file-line]] · [[project-sellsuki-product-kb]]
