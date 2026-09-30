---
name: project_oc4551_readiness_checklist
description: "OC-4551 หน้า \"ตั้งค่าให้ครบก่อนเปิดใช้งาน\" — ส่งครบทั้ง BE (!583) และ FE (!611) merged เข้า develop แล้ว 2026-09-14; unknown ปิดเกทแบบ fail-closed"
metadata: 
  node_type: memory
  type: project
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
  modified: 2026-09-30T08:39:31.626Z
---

2026-09-14 ส่งครบทั้งสองครึ่ง merged เข้า develop:
`GET /v1/company/{id}/readiness` (backoffice-api MR !583) + หน้า `/readiness`
บน backoffice FE (MR !611, commit `ae31b27`) พร้อมเมนู "ความพร้อมใช้งาน"
เป็นรายการแรกในกลุ่มการจัดการสมาชิก

9 แถว: consent_documents · otp_provider · role_permissions · point_unit ·
campaign_catalog · member_tier · staff_api_key · line_integration · company_theme

**กติกาที่ฝังเป็นโค้ด ไม่ใช่แค่คอมเมนต์:**
- `can_open_to_members` คำนวณที่ BE เท่านั้น FE ไม่ derive — เพิ่มแถว blocking
  ใหม่ทีหลังแล้วหน้าจะไม่เขียวผิด
- `unknown` ≠ `missing` คนละสี คนละข้อความ ไม่เป็นสีแดง แต่**ยังนับเป็นเหตุให้
  เกทปิด** (ไม่รู้ ≠ ปลอดภัย)
- แถว consent ไม่มีปุ่ม "ไปตั้งค่า" เพราะหน้านั้นยังบันทึกจริงไม่ได้
  (ดู [[project_oc4089_consent_binding_page]]) — แก้เป็นบรรทัดเดียวใน `FIX_ROUTE`
  ของ `ReadinessCard.vue` เมื่อพร้อม
- ~~OTP ตอบ unknown เสมอ~~ — **แก้แล้ว `394e8c7` (2026-09-22)**: ถาม CCS ได้จริง · unknown เหลือแค่ CCS ไม่ตอบ หรือผู้ถามไม่มี `sellsuki.messaging.config.view` (บริษัทเก่าอาจขาด เพราะ preset copy ตอนสร้าง)
- ~~ปุ่ม "เปิดใช้งานให้สมาชิก" ไม่มี handler~~ — **แก้แล้ว: OC-4627 merge เข้า develop ครบ 4 repo (ตรวจ 2026-09-30, สถานะ Jira "Ready to test (DEV)")** · ปุ่มทำงาน, server เช็ก `can_open_to_members` ซ้ำตอนกด, member-api อ่านสถานะผ่าน `GET /v1/company/{slug}/availability` (dev ตอบ `{"available":true|...}`, slug ไม่มีจริง 404) · ⚠️ ข้อความเก่านี้เคยถูกใช้ตอบผิดว่า "ปุ่มไม่มี handler" หลังโค้ดขึ้นแล้ว — เช็ก `git log origin/develop --grep=<card>` ก่อนอ้างสถานะ · ดู [[project_app_activation_per_company]]
- **เกทเช็กตอนกดเท่านั้น** (OC-4627 AC-24): เปิดแล้ว ถ้าแถว blocking ล้ม (เช่น OTP/เครดิต SMS) แอปยังเปิด แค่เตือนในหน้า readiness — ไม่ปิดเอง

นอก scope ที่ยังค้าง: `tier_sweep` CronJob ยังไม่ deploy — checklist บอกได้แค่ว่า
มี tier program ไม่ได้บอกว่า sweep เดินจริง
