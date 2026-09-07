---
name: reference_oc2plus_local_stack_recovery_traps
description: "Recovering the hand-started OC2Plus local stack after an overmind crash — CCS company seeding gap, stale gRPC channels, per-service env var names, zsh `path` trap, Company Owner role 65"
metadata: 
  node_type: memory
  type: reference
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-09-07T15:35:01.766Z
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

**Re-test note:** resetting OCR rows only re-runs claims still `status='pending'` (`ListPendingJobs`
joins on that); approved/rejected claims stay failed by design. Also reset `attempt_count=0`.

Related: [[project_overmind_restart_quirk]] [[reference_local_bola_own_overmind_socket]]
[[reference_keto_staging_permission_lookup]] [[project_oc4362_approve_and_admin_edit]].
