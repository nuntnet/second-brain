---
name: project_oc2plus_apikey_local_run
description: "How to run/test the OC2Plus API-key management UI locally, and the 4 gates that block it"
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-09T06:04:33.752Z
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

**🔑 AMS ตัว local = `kratos-ui-go` ที่ `accounts.sellsuki.local` (:4455) — อย่าทำ stub เอง** (ผมเคยพลาดทำ stub ก่อน user ทัก) มันเสิร์ฟ `/profile` `/login` `/logout` `/cookie` `/error` ครบ และ `/profile` คืน `{id,email,first_name,last_name}` ตรงกับที่ FE ต้องการเป๊ะ · Caddy map `accounts.sellsuki.local` → 4455 อยู่แล้ว
- **Kratos ออกแบบมาเพื่อ local SSO อยู่แล้ว:** `kratos_schema_client/kratos.yml` → `base_url: https://accounts.sellsuki.local/` · session cookie `sellsuki_session` **domain `sellsuki.local`** (แชร์ทุก subdomain) · `allowed_return_urls` ลิสต์ `https://oc2plus.sellsuki.local/` และ FE ตัวอื่นไว้แล้ว
- 🔴 **kratos-ui-go ต้องตั้ง `WHITELIST_DOMAIN`** — default `http://127.0.0.1:*` ไม่ครอบ `*.sellsuki.local` → `/cookie` เด้ง `?invalidWhitelist=false` (อาการเดียวกับ AMS dev-th ปฏิเสธ local origin) · ใช้ `{https://*.sellsuki.local*,http://localhost:*,http://127.0.0.1:*}` (gobwas/glob รองรับ `{a,b}`) · setup.sh ไม่ได้ตั้งให้ (เพิ่มแล้ว)
- ⚠️ แก้ `.env` แล้ว `overmind restart` **ไม่พอ** — air ไม่ rebuild ต้อง `touch` ไฟล์ .go ให้ air restart binary (godotenv โหลดตอน boot)

**Caddy — identity translation:** ให้ `oc2plus-api.sellsuki.local` ทำงานแทน gateway ด้วย `forward_auth localhost:4433 { uri /sessions/whoami; copy_headers X-Kratos-Authenticated-Identity-Id>X-User-Id }` → ได้ user จาก session จริง **ไม่ hardcode** · ไม่มี session = 401 จริง (FE เด้งไป login เหมือน prod) · กัน matcher `@has_user header X-User-Id *` ไว้ให้ curl/test override ได้
- 🔴 **ห้าม `import api_proxy` ข้างใน `handle`** — snippet นั้น import `common` ที่มี `tls` ซึ่งไม่ใช่ ordered HTTP handler → **config adaptation ล้มทั้ง Caddyfile ทุกโดเมนดับหมด** ไม่ใช่แค่ site นั้น · ให้ `import common` ที่ระดับ site แล้วใน handle ใส่แค่ `reverse_proxy`
- ⚠️ `handle` เป็น mutually exclusive — เอา `request_header` ไปใส่ใน handle เปล่า ๆ จะกลืน request แล้วคืน 200 body ว่าง
- FE เรียกผ่าน prefix ของ gateway: **`/backoffice/v1`** (64 จุด) และ **`/crmadmin/v1`** (apikey) → Caddy ต้อง `uri strip_prefix` ทั้งคู่
- reload ใน container ไม่เห็นไฟล์ ใช้ `docker compose up -d --force-recreate caddy` (ดู [[reference_caddy_host_networking_gotcha]])

**ยังต้องทำเอง 1 ขั้น:** `localStorage.setItem('company-id','<id>')` — หน้าเลือกบริษัทดึงจาก `/user/company` ซึ่ง CCS ตอบ `record not found` (ไม่มี mapping user↔company ใน CCS สำหรับ Kratos user นี้ · CCS ไม่มีตาราง mapping ใช้ Keto UUID-mapped tuples)

ดู [[project_oc2plus_3rdparty_apikey_gap]] · [[project_pis_frontend_local_testing]] (เคส FE ชี้ dev-th เหมือนกัน — แก้ด้วย `.env.development.local`)
