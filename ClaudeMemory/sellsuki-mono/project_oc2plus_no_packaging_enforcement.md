---
name: project-oc2plus-no-packaging-enforcement
description: OC2Plus ไม่มี client ของระบบแพ็กเกจเลย — ไม่บังคับทั้งเพดานปริมาณและการล็อกฟีเจอร์ ทุกบริษัทได้ของครบเท่ากัน
metadata:
  type: project
---

ตรวจโค้ด 2026-10-02: **OC2Plus ไม่มีตัวเชื่อมกับระบบแพ็กเกจกลางเลย**
`ls backend/oc2plus-line-crm-service-backoffice-api/src/repository | grep -iE "quota|plan|entitle"` = ว่าง
คำว่า `quota` ที่ grep เจอมีที่เดียวคือ `model.BolaQuotaCode` ตอน**ส่ง plan id ไปให้ BOLA**
ตอนสร้าง workspace — ไม่ใช่การ gate ตัวเอง

**แปลว่า** ทุกบริษัทที่เปิดใช้ OC2Plus ได้ฟีเจอร์ครบเท่ากันหมด และใช้เกินเท่าไรก็ไม่มีอะไรหยุด
ต่างจาก BOLA ที่**การนับปริมาณใช้ได้แล้ว** เหลือแค่การล็อกฟีเจอร์ (ดู [[project-sellsuki-product-kb]])

**ผลต่อการขาย:** เติมตัวเลขราคาแล้วก็ยัง**บังคับใช้ไม่ได้** จนกว่าจะทำส่วนนี้
→ ห้ามใช้ข้อจำกัดฟีเจอร์หรือเพดานเป็นจุดปิดการขาย

**อย่าสับสน 2 ชุด:**
- `oc2plus.member.view` · `oc2plus.pointclaim.review` · `oc2plus.membertier.manage` ฯลฯ (มีจริง ใช้งานอยู่)
  = **สิทธิ์ของพนักงานในร้าน** (RBAC ผ่าน Keto/rps) ละเอียดระดับฟีเจอร์
- `oc2plus.member_tier` · `oc2plus.receipt_ocr` ฯลฯ = **คีย์สิทธิ์ตามแพ็กเกจที่เสนอไว้** ใน
  `docs/product-kb/03-oc2plus.md` — **ยังไม่มีในโค้ด** ต้องเคาะราคาก่อนแล้วค่อยทำ

ต้นทุนที่ต้องคิดแยกตอนตั้งราคา: **การอ่านใบเสร็จอัตโนมัติจ่ายผู้ให้บริการภายนอกรายครั้ง**
(iApp — ดู [[project-oc4464-ocr-vendor-decision]]) ไม่ใช่ต้นทุนระบบเราเอง จึงไม่ควรเหมารวมแบบไม่จำกัด
