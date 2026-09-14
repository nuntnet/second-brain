---
name: reference_migration_files_are_not_applied_schema
description: git diff ของโฟลเดอร์ migrations ระหว่าง branch บอกแค่ว่าไฟล์อยู่ที่ไหน ไม่ได้บอกว่า DB ถูก apply หรือยัง — OC2Plus รัน migration ด้วยมือแยกจาก CI ต้องถาม DB ด้วย to_regclass ไม่ใช่ถาม git
metadata: 
  node_type: memory
  type: reference
  originSessionId: d0b39379-feaa-4fd5-b115-18617c202159
  modified: 2026-09-14T05:11:24.122Z
---

**ไฟล์ migration บน branch ≠ schema ที่ apply แล้วใน environment** (พลาดจริง 2026-09-14 ตอนปล่อย OC-4523 ขึ้น staging)

ผมเทียบ `git diff --name-only origin/main...origin/develop -- migrations/` แล้วสรุปว่า staging **ค้าง 5 migration** (017-021) เตือนผู้ใช้ถึงสามรอบว่าหน้าคูปอง/ข่าว/บัตรสมาชิก/consent จะ 500 จนกว่าจะรัน

พอรันจริงบน staging ทุกตัวขึ้น `already exists, skipping` ⇒ **schema ครบอยู่แล้วตั้งแต่แรก** คำเตือนทั้งหมดผิด

**ทำไมถึงผิด** OC2Plus รัน migration **ด้วยมือ แยกจาก CI และแยกจาก branch ของ service** (ดู [[project_oc2275_crm_migrations_run_by_hand]]) ⇒ คนอาจรันบน environment ไปแล้วนานก่อนที่ไฟล์จะถูก merge เข้า `main` การที่ไฟล์ยังไม่อยู่บน main จึงไม่ได้แปลว่า DB ยังไม่มีตาราง สองอย่างนี้เคลื่อนที่อิสระจากกันโดยสิ้นเชิง

**วิธีเช็คที่ถูก — ถาม DB ไม่ใช่ถาม git**

```sql
select to_regclass('public.<table>');   -- NULL/ว่าง = ยังไม่มี, คืนชื่อ = มีแล้ว
select count(*) from information_schema.columns
 where table_name='<t>' and column_name='<c>';   -- สำหรับ ADD COLUMN
```

**วิธีเข้าถึง DB โดยที่รหัสผ่านไม่ผ่านมือเรา** (ใช้ได้จริง ผู้ใช้รันสำเร็จ) — pod ชั่วคราวที่ดึง secret เอง

```bash
kubectl -n <ns> run crm-migrate --rm -i --restart=Never \
  --image=registry.fountain.sellsuki.com/bitnami-backup/postgresql:15.10.0-debian-12-r2 \
  --overrides="$(cat ~/crm-migrate-overrides.json)" < file.sql
```

overrides ใส่ `imagePullSecrets: [{name: sellsuki-registry}]` (จำเป็น ไม่งั้น ImagePullBackOff) และ `envFrom: [{secretRef: {name: oc2plus-crm-secret}}]`
⚠️ `kubectl run` **ไม่มี** flag `--env-from` มีแค่ `--env` ต้องใส่ผ่าน overrides เท่านั้น
⚠️ ใช้ `-i` เฉย ๆ อย่าใส่ `-t` เพราะ tty ชนกับการ redirect ไฟล์เข้า stdin
⚠️ classifier ของ harness บล็อก `kubectl run` ของเรา ⇒ ต้องให้ผู้ใช้รันเอง (ดู [[reference_harness_classifier_blocks_secrets_and_mutations]])

**สิ่งที่ช่วยให้รอด** migration ของ repo นี้เขียนเป็น `CREATE TABLE/INDEX IF NOT EXISTS` และ `ADD COLUMN IF NOT EXISTS` ทั้งหมด รันซ้ำจึงไม่ทำอะไรพัง **ตรวจ idempotency ก่อนเสนอให้รันเสมอ** และห่อด้วย `BEGIN`/`COMMIT`

เกี่ยวกับ [[reference_datastore_stale_postgres_pod]] [[feedback_verify_absence_claims]] [[project_oc4523_line_login_cards]]
