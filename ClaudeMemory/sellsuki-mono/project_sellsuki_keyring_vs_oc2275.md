---
name: project_sellsuki_keyring_vs_oc2275
description: sellsuki-keyring (PAT-2640) = central API-key service built 2026-08-18/19, AFTER OC-2275 shipped; what overlaps, what does not, and the migration cost
metadata:
  type: project
---

**repo** `sellsuki/sellsuki/backend/sellsuki-keyring` (project id 862) — fork ของ `boilerplate-backend-go` สร้าง 2026-08-17 · โค้ดจริงอยู่แต่ **branch `feature/PAT-2640-init-keyring`** · **MR !1 ยัง opened (ไม่ merge) main ยังเป็น boilerplate เปล่า** · เขียนโดย kimzey (Pan Nitiyothin) 2026-08-18/19
**Jira** PAT-2640 (In Progress) ใต้ epic **PAT-2597** (To Do) — บอร์ด **Patona ไม่ใช่ OC** · first consumer ที่ระบุคือ **channel-gateway (Python, `src/use_case/api_key.py`)** ไม่ใช่ OC2Plus · **ไม่มีการ์ด migration ของ consumer สักใบ และไม่มี link OC-2275 ↔ PAT-2597**
**ยัง deploy ไม่ได้**: keyring ไม่มี service-level auth by design → NetworkPolicy = access control ทั้งหมด แต่ helm chart ไม่มี template ให้ apply (MR !1 blocker 1) · staging อ่าน pepper ของ prod (blocker 3) · migration ไม่มี CI stage

**ทับกับ OC-2275 ตรงไหน** (OC2Plus merge เข้า develop 2026-08-10/11 = **ก่อน** keyring มีโค้ด)
- ซ้ำจริง = ฝั่ง **mint/store**: backoffice-api CRUD (!502/!518) + bcrypt + ตาราง `api_key` ใน CRM + shared `entity/apikey/secret.go` (MR !58) + OC-4432 (bcrypt cache) → keyring ใช้ **HMAC-SHA256 + Vault pepper** ไม่ใช่ bcrypt ⇒ โมดูล bcrypt ที่แชร์กันเป็นทางตัน
- **ไม่ซ้ำ (keyring จงใจไม่ทำ)**: scope catalog, per-route enforcement (OC-4424), `/.well-known/scopes` (OC-4426), Oathkeeper rule, FE UI, ตรวจว่า identity ยังอยู่จริง, `permission ∩ scope`
- **ทำถูกที่แล้ว**: verify ซ่อนหลัง interface เดียว `ApiKeyRepository.GetActiveApiKey` (`3rdparty-api/src/use_case/repository/repository.go:296-299`) ⇒ ย้ายไป keyring = เขียน repository impl ใหม่ 1 ตัว

**mismatch ที่ต้องเคาะก่อนย้าย**
1. รูปแบบ credential: OC2Plus = `X-Api-Key` + `X-Api-Secret` (`ak_...`) · keyring = token เดียว `sk_(env)_(prefix)_(secret)` ⇒ breaking กับ partner (poshmedica) + ต้องแก้ Oathkeeper `check_session_url`
2. scope ไม่ namespace ตาม app: `event.read` ผ่าน regex ของ keyring (`^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)+$`) แต่ชนกันแน่ในทะเบียนกลาง — PAT-2640 AC สั่ง "scope ต้องมี namespace" และ contract แนะให้ reuse permission string เดิม ⇒ ควรเปลี่ยนเป็น `oc2plus.*` ตอนที่ key ยังน้อย
3. backoffice-api ยังมี enum scope 10 ตัวของตัวเองบน develop (`src/use_case/model/api_key.go:13-26`) — anti-pattern เดิมที่ OC-4426 ตั้งใจล้างแต่ยังไม่ล้าง
4. `subject_kind` ของ keyring seed แค่ 7 kind แต่ **มี `sellsuki.company`** ⇒ key ของ OC2Plus เข้าโมเดลได้ตรง ๆ
5. consumer contract (`docs/consumer-contract.md` ใน branch) สั่งผู้เรียก 4 ข้อ: เช็ค identity ยังอยู่ · `effective = permission ∩ scope` (scope = เพดาน) · `active:false` ≠ 5xx · pin tenant จาก `subject` — ข้อ 1/2 OC2Plus ยังไม่ทำ

ดู [[project_oc2275_audit_actionplan]] · [[project_oc2plus_3rdparty_apikey_gap]]
