---
name: project-oc2plus-phone-stored-local-format
description: "OC2Plus เก็บ/ค้นเบอร์เป็น 0XXXXXXXXX เสมอ · API รับ 0…/66…/+66… แล้ว normalize ที่ขอบ · \"E.164\" ในการ์ด = รูปที่รับเข้า ไม่ใช่รูปที่เก็บ (PO 2026-10-01)"
metadata:
  node_type: memory
  type: project
  originSessionId: 53f9c3a5-efcb-4605-8af2-6e152327b40b
  modified: 2026-09-30T17:23:45.581Z
---

PO เคาะ 2026-10-01 (OC-4412 comment, ใช้กับ OC-4469 ด้วย): รับเข้า `0812345678` / `66812345678` / `+66812345678` = คนเดียวกัน · **เก็บและค้นเป็น `0812345678`** · ตอบกลับรูปที่เก็บ · guide (OC-4419) แนะนำ partner ส่ง E.164

**Why:** สมาชิกเดิม, OTP login `GetByPhone`, validator ของ entity และ unique `member_company_id_phone_unique` ใช้รูปนี้ทั้งหมด — เก็บเป็น +66 จะหาคนเดิมไม่เจอ/ซ้ำได้

**How to apply:** การ์ดใดเขียนว่า "normalize เป็น E.164" ให้อ่านว่ารับ E.164 ได้ แต่ normalize ลง `0…` · เบอร์ต่างประเทศยังไม่รองรับ ถ้าจะทำต้องเปลี่ยนทั้งระบบในการ์ดเดียว · ดู [[project_oc4469_oc4428_card_merge]]
