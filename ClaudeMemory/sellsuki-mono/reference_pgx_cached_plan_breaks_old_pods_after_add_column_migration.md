---
name: reference_pgx_cached_plan_breaks_old_pods_after_add_column_migration
description: "2026-10-25 chat-core ไม่ตั้ง PreferSimpleProtocol/PrepareStmt/ConnMaxLifetime → หลัง migration ADD COLUMN pod เก่าที่ cache SELECT * ไว้ได้ 'cached plan must not change result type (SQLSTATE 0A000)' ต่อ connection ต่อ statement จนกว่าจะ restart · reproduce กับ Postgres จริงใน TestAI318_Migration0116_BeforeAndAfter · เสี่ยงที่ GetByMessage (ในเส้นตอบ → rollback คำตอบ) และ conversation_goal_* (AI-314 0115) · deploy: migrate + roll pod ทุกตัวช่วงเดียวกัน · การ์ดต่อ: PreferSimpleProtocol"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
---

`cmd/chat_core_server/helper.go setupGorm` = `gorm.Open(postgres.Open(uri), &gorm.Config{})` ไม่มี option ข้างบน (DSN มาจาก secret — สมมติ pgx v5 default `cache_statement`) · pgx evict statement หลัง error ครั้งแรก จึงเป็นหนึ่ง error ต่อ connection ต่อ statement แต่ connection ไม่หมดอายุ → pod เก่าเจอทุก connection ที่ cache ไว้ · การอ่านแบบทั้ง struct (`First`/`Find` ไม่มี Select) คือ `SELECT *` = ตัวเสี่ยง; INSERT ที่ระบุคอลัมน์ไม่เสี่ยง.

**How to apply:** ทุก migration ที่เพิ่มคอลัมน์ในตารางที่อ่านด้วย SELECT * → apply แล้ว roll/restart chat-core ทุก pod ในหน้าต่างเดียวกัน (ไม่ใช่ migrate ทิ้งไว้ก่อนแล้ว deploy ทีหลัง) · เสนอแก้ถาวร: `PreferSimpleProtocol: true` หรือ explicit column list (GetByMessage ~30 คอลัมน์) · เทสที่ต้องแตะตารางก่อน/หลัง migration ต้อง reopen pool (`freshRepo`) ไม่งั้นล้มด้วยเหตุผลเดียวกัน · อย่า AutoMigrate โมเดลปัจจุบันใน migration เก่า (0071/0097 ของ decision_traces ทำแบบนั้น จนสร้างตารางก่อน 0116 ไม่ได้ → แช่แข็งโมเดล) · เกี่ยวข้อง: [[reference_gorm_pgx_libpq_automigrate]], [[reference_migration_files_are_not_applied_schema]]
