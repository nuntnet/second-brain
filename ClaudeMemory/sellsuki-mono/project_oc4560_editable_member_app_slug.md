---
name: project_oc4560_editable_member_app_slug
description: "OC-4560 — slug ของลิงก์แอปสมาชิกแก้ได้เองแล้ว: เก็บบน oc2plus_bola_bindings.slug (ไม่ใช่ company.Code), ชื่อเก่าอยู่ตลอดไปใน member_app_slug_history, และ upsert ของ binding ต้องไม่ขยับ slug"
metadata: 
  node_type: memory
  type: project
  originSessionId: cb2b2f2c-fa29-411c-9862-b07e9251a640
  modified: 2026-09-14T15:25:47.905Z
---

2026-09-14 — merchant แก้ slug ในลิงก์แอปสมาชิกได้เองจากหน้า `/member-screens` แล้ว
(3 MR: member-api !130, backoffice-api !586, backoffice FE !615 — เปิดไว้ target `develop`)

## โมเดลที่เลือก

- slug **แยกจาก `company.Code`** ของ CCS · ค่าที่ใช้จริงเก็บบน
  `oc2plus_bola_bindings.slug` · `company.Code` ไม่ถูกแตะ (ถ้าแตะจะลาก `ref_id`
  ข้ามเซอร์วิสพังตามไปด้วย)
- **ลิงก์เก่าไม่ตายตลอดไป** — ตาราง `member_app_slug_history` (migration
  `member-api/migrations/023`) PK บน `slug` · QR ที่ปริ๊นต์แปะหน้าร้าน / rich menu /
  broadcast เรียกคืนไม่ได้ จึงเลือกทางนี้แทน TTL
- ลำดับ resolve ใน `member-api ResolveCompanyBySlug`:
  **bindings → history → (Redis cache) → CCS by company code** · cache อยู่บนเส้น
  CCS เท่านั้น การเปลี่ยนชื่อจึงไม่ต้อง invalidate อะไรเลย
- คำสงวน 33 คำ อยู่ที่ `backoffice-api/src/entity/member_app_slug` · เทียบตรงตัว
  ไม่ใช่ prefix · FE มีสำเนา (กระจก) ที่ `entities/customer-app.ts` พร้อมเทสตรึงค่า

## สองกับดักที่แก้ไปแล้ว — อย่ารื้อ

**1. `Save` upsert ต้องไม่ขยับ slug**
ผู้เรียก `bolaBindingRepository.Save` **ทุกตัว** ส่ง `Slug: company.Code` ·
SQL เดิมเป็น `COALESCE(EXCLUDED.slug, stored)` → bind รอบถัดไป (retry / re-bind /
sweep) จะเขียนทับชื่อที่ merchant เพิ่งตั้ง **เงียบ ๆ ไม่ fail ไม่ log** แล้วลิงก์
ใหม่จะ 404 · ตอนนี้เป็น `COALESCE(stored, EXCLUDED.slug)` — **ลำดับ argument คือ
ตัวบั๊ก** มีเทสตรึงไว้ที่ `TestPostgresql_Save_NeverMovesAStoredSlug`

**2. `resolveCustomerApp` ต้องอ่าน binding.Slug ก่อน company.Code**
เดิมอ่าน `company.Code` ก่อน ซึ่งถูกตอนที่สองค่าขัดกันไม่ได้ · พอเปลี่ยนชื่อได้
admin จะ save ชื่อใหม่แล้วถูกแสดงลิงก์เก่า ขณะที่สมาชิกเข้าชื่อใหม่

## ที่ยังค้าง

migration `023` ยังไม่ได้รันที่ dev-th / staging / production
(ดู [[project_oc2275_crm_migrations_run_by_hand]])

เกี่ยวข้อง: [[project_customer_app_program]] · [[project_bola_binding_never_worked_via_ccs]] ·
[[reference_oc2plus_company_not_store]]
