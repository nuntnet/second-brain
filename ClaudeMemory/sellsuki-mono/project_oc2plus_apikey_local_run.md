---
name: project_oc2plus_apikey_local_run
description: "How to run/test the OC2Plus API-key management UI locally, and the 4 gates that block it"
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-09T05:31:15.759Z
---

**API-key management UI (OC-2269/2273/2274) = `frontend/oc2plus-linecrm-frontend-backoffice`** (Vue 3, :5176) → `src/views/ApiKey/{ApiKeyList,ApiKeyCreate,ApiKeyDetail}.vue`, routes `/apikey`, `/apikey/create`, `/apikey/:id`. Backend = **backoffice-api** (:8089) 4 endpoints `/v1/company/:id/api-keys`.
**ยังไม่ landed** — FE branch `feat/oc-2273-apikey-ui`, BE branch `feat/oc-2273-apikey-mgmt`.

**เทส local:** `https://oc2plus.sellsuki.local/apikey` (Caddy มีทั้ง FE และ `oc2plus-api.sellsuki.local` → :8089 อยู่แล้ว) · รัน `./scripts/seed-oc2plus-apikey.sh` (idempotent, ตรวจ 4 ด่านให้ครบแล้วบอก READY)

**4 ด่านที่บล็อกทุก company-scoped route ของ backoffice-api** (fresh local ไม่ผ่านสักด่าน):
1. **identity** — `X-User-Id` ต้อง resolve ใน **Kratos** (`GetUserProfileFromID` → kratos admin :4434) ไม่ใช่ CCS
2. **permission** — `oc2plus.apikey.manage` บน `sellsuki.company:{id}` ผ่าน **role-perm gRPC** (สร้าง role + AssignRole)
3. **DPA** — ต้องมี consentee `company_{companyID}` ที่ทุก option = accepted (`IsAllAccepted()`; **POST /consentee สร้างมาเป็น `declined` เสมอ ต้อง PUT ทับ**) · consent service ไม่มี API สร้าง `application` row ต้อง upsert ลง Mongo เอง
4. **schema** — ตาราง `api_key` **ไม่มี migration ในรีโป** (comment ในโค้ดชี้ว่าเป็นของ WS1-B ที่ยังไม่ลง) → `scripts/schema-oc2plus-crm.sql` + wired เข้า `migrate-all.sh`

**ค่า env ที่ setup.sh ตั้งผิด/ขาด (แก้แล้วทั้ง template และ .env):**
- `CCS_SERVICE_BASE_URL` เคยเป็น **8085 (ผิด — central-configuration-system)** ต้องเป็น **8092**
- `ROLE_PERMISSION_GRPC_SERVER` default `localhost:9999` ผิด — gRPC คือ **9998** และ `localhost` ไป IPv6 → ใช้ `127.0.0.1:9998`
- `CONSENT_ENDPOINT` default `:8081` = PIS → ต้อง **8096** · `DPA_CONSENT_ID` ต้องตรง consent id จริง
- consent service **ไม่ได้ตั้ง `KAFKA_TOPIC_EVENT_*`** → ทุก write ตอบ `event_firing_failed` (publish ก่อน commit)

**บั๊กที่ทำให้ service ไม่ขึ้นเลย:**
- **backoffice-api ไม่มี `.air.toml`** → air build root ที่ไม่มี main package → 502 (สร้างให้แล้ว)
- Kafka consumer ถูกสร้าง**เสมอ ไม่เช็ค `KAFKA_ENABLED`** → panic `either Topic or GroupTopics must be specified` ถ้า `KAFKA_TOPIC_CAMPAIGN_STATUS_COMMAND` ว่าง
- CCS boot health-check **fail-fast** ต้องมี sukipay gRPC :50070 + address gRPC :50058 ขึ้นก่อน ไม่งั้น CCS ตาย

**🔴 บั๊กในโค้ด OC-2273 ที่เจอและแก้ (ยังไม่ commit):** `api_key_repository/postgresql.go` `scanApiKeyRows` scan **9 ปลายทาง แต่ `apiKeyColumns` select 10** และข้าม `key` → list/detail 500 เสมอ ("expected 10 destination arguments in Scan, not 9") — **ฟีเจอร์ใช้ไม่ได้จริงบน branch นั้น**

**Caddy:** เพิ่ม local-dev identity shim ที่ `oc2plus-api.sellsuki.local` (inject `X-User-Id` เมื่อ caller ไม่ได้ส่ง) เพราะ FE ส่ง `Authorization: Bearer` แต่ backend อ่าน `X-User-Id` — dev-th มี gateway แปลงให้ local ไม่มี · ⚠️ ต้องใช้ `request_header @matcher` **ห้ามใส่ใน `handle`** (handle เป็น mutually exclusive → คืน 200 body ว่าง) · reload ใน container ไม่เห็นไฟล์ ใช้ `docker compose up -d --force-recreate caddy` (ดู [[reference_caddy_host_networking_gotcha]])

ดู [[project_oc2plus_3rdparty_apikey_gap]] · [[project_pis_frontend_local_testing]] (เคส FE ชี้ dev-th เหมือนกัน — แก้ด้วย `.env.development.local`)
