---
name: reference_oc2plus_local_stack_recovery_traps
description: "Recovering the hand-started OC2Plus local stack after an overmind crash — CCS company seeding gap, stale gRPC channels, per-service env var names, zsh `path` trap, Company Owner role 65"
metadata: 
  node_type: memory
  type: reference
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-09-08T05:33:11.208Z
---

**Context (2026-09-07):** root overmind died; OC2Plus services were hand-started. A browser console
showing `500 /company/{id}`, `403 /capabilities`+`/themes`, `401 portal-app-registry`, "OCR ไม่ทำงาน"
had FOUR independent root causes — none was the obvious one.

**1. CCS `GET /company/11111111…` → "record not found" (surfaced as 500).** The OC2Plus local test
company `11111111-1111-4111-8111-111111111111` exists in `oc2plus_crm` + Keto but was **never
registered in CCS's `sellsuki_central_control.companies`** (seeding gap — seed-dev creates its own
"Sukispace" company via gRPC, not this fixed id). `CompanyService/CreateCompany` PANICS (nil deref,
needs sellsuki-pay :50070 up) so seed by SQL instead:
`INSERT INTO companies (id, provider_code, company_code, name, phones, address_country_code,
tax_address_country_code) VALUES ('11111111-…','sellsuki','localtest','Local Test Company',
'{0000000000}','THA','THA') ON CONFLICT (id) DO NOTHING;` — connect with CCS's `POSTGRES_URI`
(a full URI; run `docker exec sellsuki_mono-postgres-1 psql "$POSTGRES_URI"`). NOT NULL = id,
provider_code, company_code only.

**2. Stale gRPC channel — restart ORDER matters.** After the insert CCS still 500'd with
`dial tcp 127.0.0.1:50058 refused` even though address-backend was LISTENING and `grpcurl
127.0.0.1:50058 list` worked. CCS had built its gRPC client at boot while address-backend was down
and never recovered the channel. **Fix = restart CCS AFTER its gRPC deps are up** (address-backend
:50058, sellsuki-pay :50070). Symptom signature: "connection refused" to a port that lsof/grpcurl
prove is open → it is the CALLER's stale channel, not the server.

**3. Per-service env var names differ — read the Procfile line, don't assume.** address-backend
uses `GRPC_LISTEN_ADDRESS=:50058` (NOT `GRPC_PORT`); CCS uses `GRPC_PORT=50060`; i18n/kratos-ui take
`PORT` only. Starting with the wrong name binds the default port silently. Always copy the exact
Procfile entry: `W=scripts/svc-start.sh; cd backend/<svc> && set -a; . ./.env; set +a; PORT=… "$W" <http> <grpc>`.
(No godotenv in these Go services — env must be in the process; see [[project_oc4362_approve_and_admin_edit]].)

**4. zsh: never use `path` as a loop variable.** `for path in …` in zsh assigns the special array
bound to `$PATH` → every subsequent command is "command not found". Use `ep`/`p`.

**Permissions: "Company Owner" = rps role 65.** Keto model here = namespace `permissions`, object =
permission string, relation = `sellsuki.company:<id>`, subject = `sellsuki.user:<id>`. The user had
only 3 direct tuples (company.view/update, pointclaim.review) → 403 on capabilities/themes. Proper
grant is via rps gRPC (`localhost:9998 role_and_permission.RoleAndPermissionService`):
`CreateRole{name, permissions[], owner:{id:<company>,kind:sellsuki.company}, actor}` →
`AssignRole{role_id, tenant:{kind:sellsuki.company,id}, user:{id,kind:sellsuki.user}, actor}`.
Role **65 "Company Owner"** on company 11111111 = all 12 `oc2plus.*` + sellsuki company-scoped
(company.*, user.*, role.*, filesystem.*, consent.manage, payment.*). Full catalogue lives in
`gitlab.sellsuki.com/sellsuki/sellsuki/backend/entity/access_control` (go mod cache). ListRoles uses
`filter_options.owner_id/owner_kind` + `list_options.page/limit`. Kind MUST be prefixed
([[reference_rps_identity_kind_must_be_prefixed]]).

**`401 portal-app-registry` (central-config via Caddy) is cosmetic.** Direct `:8085` with identity
headers → 200 but `apps: []` anyway; via `https://central-config.sellsuki.local` → 401 "No valid
session credentials" = Kratos forward_auth at the proxy not seeing the cross-subdomain session cookie.
Fixing it yields an empty app switcher. Don't sink time.

**"OCR ไม่ทำงาน" was NOT config.** After the vlm fix, re-running failed rows: real receipts → partial
with items; 2 claims → Gemini **HTTP 400** because their attachments are degenerate TEST images
(file 11 = 1×1 PNG 147 B, file 23 = 22-byte JPEG stub). Worker lumps provider 400 (bad input,
permanent) with 5xx (outage) as one `PROVIDER_ERROR` → admin can't tell "photo unusable, ask member
to resend" from "OCR down". Product fix belongs in the review dialog: classify 400→"รูปอ่านไม่ได้",
5xx/timeout→"ระบบ OCR ขัดข้อง", 401/403→"ตั้งค่า OCR ผิด". Fetch an attachment like the worker:
`GET $FILE_SERVICE_URL/access/private?refID=sellsuki.company:<id>&fileRecordID=<n>` with
`X-User-Id: $FILE_SERVICE_API_KEY`, `X-User-Kind: sellsuki.system`.

**Backoffice FE: "เมนูคำขอแต้มหาย" after granting MORE permissions = permission-store race, not Keto.**
`stores/permission` was a single slot (`permission.value = null` then `= <this call's results>`) shared by
19 `usePermissions()` askers (App.vue panel gate, SideBar, mounted view) → last resolver erased the others'
keys → `GetPermissionStatus('oc2plus.pointclaim.review')` false by timing. Broader grants = more askers =
worse. Backend was fine (`POST /v1/company/{id}/permission` → is_allow:true). Fixed `ba13a2d` (!572):
merged `Record<name,bool>`, in-flight counter for `loading`, `SetGrants()` for tests. Signature: a menu
that flickers/vanishes while `POST …/permission` says true.

**Backoffice FE company picker is a LOCAL STUB, not the backend.** `vite.config.ts` (UNCOMMITTED, the
user's) has `localCompanyListPlugin` answering `GET /backoffice/v1/user/company` with company `localtest`
and `localProfilePlugin` answering `/profile` as "Nuntawit (local)". Requests never reach :8089. An
"empty picker" in the user's Chrome while the pane shows the company = the user's tab is a stale bundle
from before a Vite restart (old `VITE_SERVICE_BACKOFFICE_BASE_URL` → dev-th → 401 → caught → `[]`);
hard-reload fixes it. `.env.development.local` (highest priority) holds the local overrides.

**Theme logo broken locally = wrong file host, not the theme.** `GET /company/{slug}/theme` returns
`logo` as an S3 OBJECT KEY (`/<company-uuid>/public/<ts>-logo-<ts>.png`); member-FE `services/theme`
runs it through `utils/file-url.ts` `toFileUrl()` = `VITE_FILE_SERVICE_BASE_URL + key`. Default env points
at `https://file.dev-th.sellsuki.com` (cloud) but the upload went to LOCAL file-service → MinIO
(`sellsuki_mono-minio-1` :9000, bucket `file-service`). file-service :8087 does NOT serve public objects
(all paths 404) and the bucket policy was `private` (anon GET 403). Fix (local only): both FEs'
`.env.development.local` → `VITE_FILE_SERVICE_BASE_URL=http://localhost:9000/file-service`; MinIO anonymous
read scoped to `arn:aws:s3:::file-service/*/public/*` via `docker exec sellsuki_mono-minio-1 sh -c 'mc alias
set local http://localhost:9000 "$MINIO_ROOT_USER" "$MINIO_ROOT_PASSWORD"; mc anonymous set-json <policy>
local/file-service'` (creds never leave the container; the minio shell has no grep/awk — filter on host).
Verified: logo 200, a `/private/…rc.jpg` receipt still 403. Pre-existing: uploads are stored as
`application/octet-stream` (file-service ignores X-Original-Content-Type) — browsers sniff `<img>` fine.

**App switcher "ไม่แสดงชื่อ app" = the registry fetch, not the data.** `portal-app-registry` (central-config)
at version 1 already holds Sellsuki/Akita/Patona/Oc2plus (icons on `assets.staging.sellsuki.com`); it read
`apps: []`/version 0 only while central-config was freshly restarted. The FE hit
`https://central-config.sellsuki.local/v1` → Kratos forward_auth 401 (Chrome) / ERR_NAME_NOT_RESOLVED
(pane) → store error → no tiles. Fix (local): `vite.config.ts` proxy `/central-config` → :8085 with the
same injected X-User-Id/Kind as `/backoffice`, and `.env.development.local`
`VITE_CENTRAL_CONFIGURATION_SYSTEM_URL=/central-config/v1`. Writing a GLOBAL config needs
`sellsuki.configsystem.config.update` on tenant `sellsuki.user:` (EMPTY id) via a role (rps roles 61/62
are the existing global viewers) — created role **66 "Local Config Admin"** for that; my ad-hoc PUT failed
schema validation (400) and was not needed. `PUT /v1/configuration/{service}?userId=&location=` body
`{"data":{…}}`, validated against the seeded schema (0002).

**FORMALIZED 2026-09-08 — the local-dev setup is now committed, not per-machine:**
- backoffice FE `013a38f` (!572) / member FE `5f919fa` (!35): `vite.config.ts` reads `.env.development.local`
  via `loadEnv`; the stub/proxy layer is gated on `VITE_LOCAL_DEV_IDENTITY_ID` (backoffice) /
  `VITE_LOCAL_DEV_MEMBER_API_TARGET` (member), OTP test header from `VITE_LOCAL_DEV_TEST_SECRET` (must equal
  member-api `TEST_KEY`; verified via Redis `otp_session…` `"IsTestMode":true`). Committed
  `.env.development.local.example` in both; real `.env.development.local` stays gitignored (`*.local`).
- monorepo `38e7e49`: `scripts/seed-dev.sh` section **4c** = CCS `local_sellsuki_central.companies` row for
  11111111 (NOT `sellsuki_central`, which is empty), `oc2plus_crm.oc2plus_bola_bindings` slug row, rps roles
  **"OC2Plus Company Owner (local dev)"** (12 oc2plus.* + sellsuki company-scoped, owner=company) and **"Local
  Config Admin (user-scope)"** (configsystem.config.*, owner `sellsuki.user:""`), MinIO anonymous read on
  `file-service/*/public/*` via `docker compose exec -T minio mc anonymous set-json`. `find_or_create_role`
  matches by name under owner (needs `jq`) → idempotent; my hand-made roles 65/66 were RENAMED to those
  names so the seed finds them. `8a2dfa4`: CLAUDE.md points `make seed-dev` at
  `.claude/knowledge/local-dev-oc2plus.md` (the end-to-end guide).
- ⚠️ `DEV_USER_ID` default in seed-dev.sh is `d46da77b-5f10-479a-9006-f91939a0b85f`, NOT the user's real
  Kratos id `317c3e2a-…` — always run `DEV_USER_ID=<id> make seed-dev`. Monorepo has NO origin; its only
  remote `glab-base` is the BOLA repo — never push there.

**seed-dev.sh was lying until 2026-09-08 — the full-run audit found 7 false-success paths:** bare host
`psql` absent → every SQL step no-op'd but printed ✓ (now `psql_db` = compose exec); Kafka container
hardcoded `repos-kafka-1` (real `sellsuki_mono-kafka-1`) → 5/6 topics never existed; provider/company
SQL aimed at empty `sellsuki_central` (CCS uses `local_sellsuki_central`); store INSERT aimed at db
`postgres` (table is space-go's `space.store`); quota `InternalCreateQuota` payload predated the
wrapped proto (`{"quota":{owner{id,kind:"sellsuki.provider"},…,state:<JSON string>,usage_flow}}`)
and exit code unchecked → 6 plans "(created)" that didn't exist; config-schema POST went via Caddy →
Kratos 401/403 reported as "central-config down" (now direct `:8085` + identity headers); and the
big one — 5 role blocks assigned ONLY when CreateRole succeeded, so with roles pre-existing a new
`DEV_USER_ID` got nothing (now `grant_role` = find-or-create + assign; rps dup-assign error
"user already has this role in this tenant" = ok). CCS `HealthCheck` hangs when sellsuki-pay :50070
is down — readiness probe is now `grpcurl list`. Lesson: a seed that swallows stderr and prints a
fixed success string is worse than no seed; verify by exit code, re-run twice, diff rps roles.

**Re-test note:** resetting OCR rows only re-runs claims still `status='pending'` (`ListPendingJobs`
joins on that); approved/rejected claims stay failed by design. Also reset `attempt_count=0`.

Related: [[project_overmind_restart_quirk]] [[reference_local_bola_own_overmind_socket]]
[[reference_keto_staging_permission_lookup]] [[project_oc4362_approve_and_admin_edit]].
