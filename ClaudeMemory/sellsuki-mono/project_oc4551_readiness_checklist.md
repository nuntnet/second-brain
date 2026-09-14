---
name: project_oc4551_readiness_checklist
description: "OC-4551 หน้า \"ตั้งค่าให้ครบก่อนเปิดใช้งาน\" — ส่งครบทั้ง BE (!583) และ FE (!611) merged เข้า develop แล้ว 2026-09-14; unknown ปิดเกทแบบ fail-closed"
metadata: 
  node_type: memory
  type: project
  originSessionId: fa3d17a9-10e1-44ea-8608-642094bcd0f3
  modified: 2026-09-14T12:00:32.646Z
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
- OTP ตอบ `unknown` + reason `no_readiness_endpoint` เพราะ messaging-backend
  มีแค่ request/verify ไม่มีทางอ่านสถานะ config โดยไม่ส่ง SMS จริง

นอก scope ที่ยังค้าง: `tier_sweep` CronJob ยังไม่ deploy — checklist บอกได้แค่ว่า
มี tier program ไม่ได้บอกว่า sweep เดินจริง
