---
name: project_oc2275_audit_actionplan
description: OC-2275 API-key audit 2026-08-09 (repo/DDD/branch/MR) + แผนแก้ 4 ข้อที่ user อนุมัติแล้ว — ยังไม่ได้ลงมือ
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-09T17:16:44.470Z
---

**Audit เต็มของ OC-2275 API-key workstream (2026-08-09) — user อ่านแล้วบอกว่า "วิเคราะห์ถูกต้องทั้งหมด อยากทำทั้งหมด" แต่ token ไม่พอ จึงบันทึกไว้ทำต่อ**
สำเนาสำหรับทีมอยู่ที่ **comment ของ OC-2275** (ดู index คอมเมนต์ด้านล่าง) — ถ้าไฟล์นี้หายให้ไปอ่านที่นั่น

## สถานะ ณ 2026-08-09: ยังไม่มีอะไร merge เลยสักรีโป (ไม่มี merge ผิดที่)

| repo | MR | source → target | สถานะ |
|---|---|---|---|
| 3rdparty-api | !207 | `feat/oc-2273-apikey-crud` → develop ✅ | Draft |
| backoffice-api | !502 | `feat/oc-2273-apikey-mgmt` → develop ✅ | Draft |
| FE linecrm-backoffice | !527 | `feat/oc-2273-apikey-ui` → develop ✅ | Draft |
| member-api | !71 | 🔴 `feat/oc-2273-apikey-migration` → **`feat/oc-4289-post-members`** | ไม่ใช่ draft |

## 🔴 งานที่ user อนุมัติแล้ว — ยังไม่ได้ลงมือ (ทำต่อจากตรงนี้)

### 1. WS1-B migration ย้ายรีโป + ปิด MR !71 ✅ อนุมัติ
- ผิดยังไง: WS1-B อยู่ที่ `member-api/migrations/003_add_api_key_columns.up.sql` แต่**บ้านจริงของ schema CRM คือรีโป `git@gitlab.sellsuki.com:sellsuki/oc2plus/line-crm/migration/oc2plus-line-crm-migration.git`** ซึ่งเป็นที่ที่ `api_key` ถูกสร้าง (`20250204031115-create-table-api-key-up.sql`)
- คอมเมนต์ในไฟล์เขียนเหตุผลผิดว่า member-api คือ "CRM DB's version-controlled migration set" — ไม่จริง · member-api ไม่มี runner (ไฟล์เขียนเองว่า "no automated runner in this repo") · เลข 001/002/003 คนละระบบกับ db-migrate timestamp
- MR !71 target `feat/oc-4289-post-members` ซึ่ง **ยังไม่ merge (+3 จาก develop) และไม่มี MR ของตัวเอง** → WS1-B ไปไม่ถึง develop ทางนี้ · branch ยังลาก 3 commit ของ OC-4289 (LINE-agnostic members + OTP) ติดมา
## ✅ ข้อ 1 ทำเสร็จแล้ว 2026-08-10

- **migration repo !73** (Draft) `feat/oc-2273-ws1b-api-key-management-columns` → **develop** — WS1-B ฉบับย้ายบ้าน + แก้ `key varchar(32)`→64 · **merge หลัง !72 เท่านั้น**
- **migration repo !72** `chore/sync-develop-with-main` → develop — 🔴 **main กับ develop ของรีโปนี้ unrelated histories** (main = 1 commit squashed 2026-07-31 มี 77 migrations · develop = history เดิมปี 2024 มี 6 migrations, ไม่มีไฟล์ไหนที่ main ไม่มี, ต่างแค่ README+package.json) → merge `--allow-unrelated-histories` เอาฝั่ง main, tree เท่ากับ main เป๊ะ, ไม่ force push
- **member-api !71 ปิดแล้ว** พร้อมคอมเมนต์ชี้ทางไป !72/!73 (note_83991)
- ⏳ เหลือ: **ลบ `member-api/migrations/003_add_api_key_columns.{up,down}.sql`** ไม่ให้มี DDL ของ api_key สองชุด

**▶ ความคืบหน้าเดิม 2026-08-09:** สร้าง+push แล้ว branch `feat/oc-2273-ws1b-api-key-management-columns` ในรีโป migration (commit `58a96c7`, 3 ไฟล์: `20260809000000-alter-table-api-key-add-management-columns` .js + sqls up/down) · verify แล้วว่า up.sql รันผ่านบน local oc2plus_crm และ idempotent
🔴 **ยังไม่เปิด MR — ติดคำถาม target branch**: รีโป migration นี้ **`develop` ตายตั้งแต่ 2024-03-07 (6 migrations) ส่วน `main` คือของจริง (77 migrations, active 2026-07-31)** → ขัดกฎ "OC2Plus merge to develop เท่านั้น" ต้องให้ user เคาะก่อน
⚠️ **ตัดสินใจระหว่างทาง**: `expires_at` ใช้ **timestamptz** (ไม่ตาม convention `without time zone` ของตาราง) เพราะ GetActiveApiKey เทียบคอลัมน์กับ Go `time.Time` ที่ driver ส่งเป็น timestamptz → ถ้าเป็น without-tz Postgres จะ cast ด้วย session TimeZone แล้วอ่าน UTC เป็น Asia/Bangkok = **เพี้ยน 7 ชม.**
ยังค้าง: เปิด MR · ปิด !71 · ลบไฟล์ออกจาก member-api
- **สิ่งที่ต้องทำเดิม**: สร้าง migration ในรีโป migration ด้วยชื่อ db-migrate timestamp (`YYYYMMDDHHMMSS-alter-table-api-key-*.sql` up+down) เนื้อหา = ALTER 3 คอลัมน์ + partial unique index **+ `ALTER TABLE api_key ALTER COLUMN key TYPE varchar(64)`** (บั๊ก key 35 ตัวเกิน varchar(32)) → เปิด MR ที่รีโปนั้น → ปิด !71 พร้อมคอมเมนต์ชี้ไป MR ใหม่ → ลบไฟล์ออกจาก member-api

### 2. bcrypt/scope ซ้ำ 2 รีโป — **คำตอบที่ให้ user แล้ว**
ทั้ง 3 รีโปใช้ shared module `gitlab.sellsuki.com/.../line-crm/backend/entity` อยู่แล้ว (backoffice pin v1.7.3, 3rdparty pin v1.7.1)
- **bcrypt hash/verify → ย้ายเข้า shared entity module** (ที่เดียว) · เป็น pure function ที่สองฝั่งต้องตรงกัน byte-for-byte · วันนี้ซ้ำอยู่ 2 ที่ (`api_key_repository/secret.go` ทั้ง backoffice=hash และ 3rdparty=verify) ผูกกันด้วย**คอมเมนต์อย่างเดียว** ("changing it here breaks cross-service auth") · ข้อดีของ shared module: version skew กลายเป็น compile error ไม่ใช่ drift เงียบ
- **scope catalog → ห้ามเอาเข้า shared module** (version skew = drift เงียบอีกแบบ) → **3rdparty-api เป็นเจ้าของ + ประกาศผ่าน registry `/.well-known/scopes` (OC-4426)** และ **backoffice-api เลิกมี enum ของตัวเอง** · ⇒ OC-4426 กลายเป็น blocker ของการลบ enum ซ้ำ

### 3. แก้ OC-2273 ตัด `apikey.manage` ออกจากตาราง scope ✅ อนุมัติ
การ์ดยังลิสต์เป็น scope ที่ 11 + บอกว่า gate ทั้ง 4 endpoint · โค้ดจริงหลัง refactor **ตัดออกจาก catalog โดยตั้งใจ** แล้ว gate ด้วย session/company permission แทน (เขียนไว้ใน `use_case/model/api_key.go` comment) → **แก้การ์ด ไม่ใช่แก้โค้ด**

### 4. FE อยู่ที่เดิม (CRM backoffice) ✅ user เห็นด้วย — บันทึกเป็น decision ใน OC-2275
เหตุผล: CCS3 = tenant admin (Company/User/UserGroup/Consent/Pay/Product/Location) ไม่แตะ CRM · key ชุดนี้ให้สิทธิ์ CRM domain ล้วน + permission ชื่อ `oc2plus.apikey.manage` · **ย้ายเมื่อ** central key service เกิดจริง **และ** key ครอบ >1 domain

## ข้อมูลอ้างอิงที่ใช้ตอน audit (กันต้องไปขุดใหม่)

- **scope catalog อยู่ 4 ที่**: backoffice-api model=10 (ฝั่ง mint) · 3rdparty-api model=3 (ฝั่ง enforce) · FE `entities/apikey.ts`=10 · การ์ด=11 → ฝั่งแจก 10 ฝั่งบังคับ 3 (ไม่ใช่ privilege escalation แต่เป็นสัญญาที่ทำตามไม่ได้)
- **DDD layering ถูก** (interface → use_case → repository, model ใน use_case/model, mock ใน use_case/repository) · ที่ผิดคือ bcrypt อยู่ใน repository layer + ซ้ำ 2 รีโป
- ⚠️ **backoffice-api (บ้านชั่วคราวของ CRUD) มี enum ของ scope เอง = anti-pattern เดียวกับที่เคาะว่า central ห้ามทำ** — ห้ามยกโค้ดนี้ไป central ตรง ๆ
- FE commit `9f45c1b` message ยังเขียน "Store Admin" (push แล้ว แก้ไม่ได้ — MR title แก้ได้)
- CCS3 มี `lib/entities/store/store.ts` = stub 55 bytes `{id,name}` (ไม่เกี่ยวกับงานนี้)

## Index คอมเมนต์บน OC-2275 (บริบทสะสมทั้งหมด)
`43431` เส้นแบ่ง central↔service · `43433` แก้ข้อมูลที่ผมเคยสรุปผิด (scope **enforce จริง** 4/4) · `43436` hot path/SPOF/bcrypt 47ms/JWKS · `43438` terminology company ไม่ใช่ store
บน OC-2273: `43437` เตือน reviewer MR !207 เรื่อง bcrypt ต้องมี cache ก่อน merge
บน OC-4398: `43432` ถอนคำแย้งของผม (การ์ดถูก)

ดู [[project_oc2plus_3rdparty_apikey_gap]] · [[reference_oc2plus_company_not_store]] · [[feedback_verify_absence_claims]] · [[project_oc2plus_apikey_local_run]]
