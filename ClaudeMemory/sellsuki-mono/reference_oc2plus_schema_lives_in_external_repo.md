---
name: reference_oc2plus_schema_lives_in_external_repo
description: OC2Plus CRM schema อยู่ repo migration แยก (project 530) ที่ไม่ใช่ submodule — member-api/migrations ไม่มี runner ไฟล์ที่อยู่แค่ที่นั่นไม่มีวันรัน
metadata:
  node_type: memory
  type: reference
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-09-17T23:05:00.000Z
---

**ไม่มี OC2Plus CRM service repo ไหนถือ migration ของตัวเอง** — ไม่มี goose/golang-migrate/
AutoMigrate ใน backoffice-api / 3rdparty-api / member-api เลย ทั้งที่ query goqu อ้างตารางเพียบ

schema จริงอยู่ที่ **repo แยกที่ไม่ใช่ submodule**:
`sellsuki/oc2plus/line-crm/migration/oc2plus-line-crm-migration` · **GitLab project 530**
clone ตรงได้ด้วย glab/https · `scripts/migrate-all.sh:140-160` clone on-demand ลง `.cache/`
**applied by hand ต่อ environment ไม่มี automated runner ไม่มี CI** — merge แล้วไม่ได้แปลว่ารันแล้ว

รูปแบบ: db-migrate · `migrations/YYYYMMDDHHMMSS-<ชื่อ>.js` (wrapper อ่านไฟล์)
\+ `migrations/sqls/<ชื่อเดียวกัน>-up.sql` และ `-down.sql` · copy ใบล่าสุดมาแก้ชื่อ

🔴 **กับดักที่คนเดินซ้ำอย่างน้อยสองรอบ (ผมเป็นรอบที่สอง 2026-09-17):**
`backend/oc2plus-line-crm-service-member-api/migrations/` **ไม่ใช่ชุดที่รัน** —
มันไม่มี runner เลย ไฟล์ที่อยู่แค่ที่นั่นจะไม่มีวันถูก apply ที่ไหน
- หัวไฟล์ `20260809000000-alter-table-api-key-add-management-columns-up.sql` ในรีโป 530
  เขียนเตือนไว้เอง ว่ารอบแรกมีคนเอา statement ไปใส่ `member-api/migrations/003_add_api_key_columns.up.sql`
  เพราะเชื่อว่าที่นั่นคือชุด migration ของ CRM — **มันไม่ใช่**
- `docs/release/environments.yaml` บันทึกตอนจบ: 4 migration อยู่แต่ใน member-api
  ไม่เคยเข้ารีโป 530 → dev ตอบ 500 สองหน้า *"they were not pending, they were absent"*

⚠️ **การ grep ว่า "ตารางนี้ไม่มี migration" ต้อง grep ที่ project 530 ไม่ใช่ member-api**
ผมเคยรายงานผิดใน OC-4567 ว่า `api_key` ไม่มีไฟล์ DDL ทั้งที่มีทั้ง create (2025) และ
alter ของ OC-2273 merge เข้า main ไปแล้ว — เพราะ grep ผิดรีโป

**เลข/ชื่อชนกันได้:** member-api/migrations นับ 004–025 ส่วนรีโป 530 ใช้ timestamp
บางใบมีในรีโป 530 แต่ไม่มีใน member-api และกลับกัน — เช็คทั้งสองที่ก่อนตั้งชื่อ

**timestamptz ไม่ใช่ timestamp สำหรับตารางใหม่** — ตารางเก่า (api_key, session, login)
เป็น `timestamp without time zone` แต่ทุกตารางที่เพิ่มตั้งแต่ 2026-08 (news, coupon,
company_consent, member_card_token, api_key.expires_at) ใช้ timestamptz โดยตั้งใจ
เหตุผลเขียนไว้ในหัวไฟล์ api_key: driver ส่ง Go time.Time เป็น timestamptz การเทียบกับ
คอลัมน์ไร้ timezone ทำให้ Postgres cast ด้วย session TimeZone → เพี้ยนไป 7 ชั่วโมง
ดู [[reference_naive_timestamp_columns_shift_by_host_tz]]

ดู [[reference_migration_files_are_not_applied_schema]], [[project_oc2275_crm_migrations_run_by_hand]],
[[project_oc4362_claim_cluster_gaps]]
