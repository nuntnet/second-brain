---
name: feedback-verify-as-the-user-sees-it
description: ห้ามบอกว่า publish/deploy เสร็จจาก tool log หรือ HTTP 200 — ต้องเปิดดูในสภาพที่ user จะเห็นจริง (session ที่ล็อกอิน)
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 60159299-7c45-49ce-8985-bacd71095aa0
  modified: 2026-08-14T11:01:04.286Z
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

**ส่วนขยาย (2026-08-14) — "200 ทุกตัว" ไม่ใช่หลักฐานว่าไม่มีบั๊ก:** เปิด admin UI ใน Chrome ที่
ล็อกอินจริง เจอสองบั๊กที่ curl/HTTP-200 ให้ผ่านทั้งคู่ (ก) จอแรกขึ้น error แต่กด retry แล้วได้ —
race ของลำดับ render/effect ที่ยิง request แรกไปแบบไม่มี scope header (ข) **หน้า idle ยิง 164
request ใน 10 วินาที** — 200 หมด ไม่มี error ไม่มีจอไหนดูผิด มองไม่เห็นเลยถ้าไม่เปิด network

**How to apply เพิ่ม:**
5. ตอนเปิดหน้าใน browser ให้ **วัด request rate ตอนหน้านิ่ง ๆ** (clear network → รอ ~10 วิ → นับ)
   ไม่ใช่ดูแค่ว่า request สำเร็จ — refetch storm มีหน้าตาเป็น 200 ล้วน
6. ทดสอบ **cold load** เสมอ ไม่ใช่แค่หลังกด retry — บั๊กลำดับ (effect vs render) โผล่แค่ครั้งแรก
7. 401 ที่เราสร้างเอง แยกไม่ออกจาก session หมดอายุ ถ้าเจอเด้งไปหน้า login ให้สงสัย
   "เราส่ง header ไม่ครบ" ก่อนสงสัย cookie

เกี่ยวข้อง: [[reference-outline-mcp-vpn-blocker]] (กับดัก 3 ข้อของ Outline MCP) ·
[[feedback-ground-claims-file-line]] · [[project-sellsuki-product-kb]] ·
[[reference_browser_surfaces_this_workspace]]
