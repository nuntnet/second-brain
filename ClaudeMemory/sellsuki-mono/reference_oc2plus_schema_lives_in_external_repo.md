---
name: reference_oc2plus_schema_lives_in_external_repo
description: OC2Plus CRM schema ไม่ได้อยู่ใน service repo ใดเลย — อยู่ repo migration แยกที่ไม่ใช่ submodule
metadata: 
  node_type: memory
  type: reference
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-08-27T01:54:39.278Z
---

**ไม่มี OC2Plus CRM service repo ไหนถือ migration ของตัวเอง** — ไม่มี goose/golang-migrate/AutoMigrate/.sql
ใน `oc2plus-line-crm-service-backoffice-api` และ `-3rdparty-api` เลย ทั้งที่ query goqu อ้างตารางเพียบ

schema จริง (78 migrations, ~49 ตาราง) อยู่ที่ **repo แยกที่ไม่ใช่ submodule**:
`git@gitlab.sellsuki.com:sellsuki/oc2plus/line-crm/migration/oc2plus-line-crm-migration.git` (db-migrate, npx)
→ `scripts/migrate-all.sh:140-160` clone on-demand ลง `.cache/` · **applied by DBA ในของจริง ไม่มี automated runner**

**วิธีเพิ่ม DDL ที่ migration repo ยังไม่รับ (precedent ที่ใช้จริง — OC-2273 WS1-B):**
เขียนเป็นไฟล์ numbered `.up.sql` + `.down.sql` ใน
`backend/oc2plus-line-crm-service-member-api/migrations/` — dir นี้คือ "the CRM DB's
version-controlled migration set" (ระบุเองที่ `003_add_api_key_columns.up.sql:1-10`)
มีแล้ว 001–003 → ไฟล์ถัดไปคือ **004** · ต้อง additive + idempotent (`IF NOT EXISTS`)
`scripts/migrate-all.sh:162-177` apply ให้ตอน local dev ผ่าน `scripts/schema-oc2plus-crm.sql`

⚠️ แปลว่าการ "freeze schema" ของ feature ใหม่ = ต้องแตะ repo ที่ 3 หรือใช้ precedent นี้ —
ไม่ใช่ `make migration` ในรีโปตัวเอง

ดู [[project_oc4362_claim_cluster_gaps]], [[reference_oc2plus_company_not_store]]
