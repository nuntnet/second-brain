---
name: project_oc4348_web_otp_session_minter
description: OC-4348 phone+OTP web login — the session-minter gap in member-api and the build approach decided 2026-09-07
metadata: 
  node_type: memory
  type: project
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-09-07T01:41:22.554Z
---

**OC-4348** = "[Customer App] Web/Desktop login (non-LIFF, OTP) — MVP". The
customer-facing 401-after-OTP bug traces here: member-api today mints an
`oc2plus_crm_session` **only via LINE** (`LineLogin`), so phone+OTP register
(OC-4245) verifies identity but never mints an app session → subsequent
`POST /v1/me/point-claims` 401s. See [[project_oc2plus_customer_app_auth_plan]].

## Key member-api auth architecture (verified 2026-09-07)
- **No auth middleware.** Each protected handler reads cookie `oc2plus_crm_session`
  inline, then `resolveMemberSession(ctx, token)` in `src/use_case/point_claim.go`
  does: `GetActiveSessionByToken` → `integrationRepository.GetByID(sess.IntegrationID)`
  → returns `(inte.CompanyID, sess.MemberID)`. **Company is derived from the LINE
  integration, not stored on the session.**
- `session` table (`src/repository/session_repository/postgresql.go`) has
  `integration_id` but **NO `company_id`**. Entity `session.Session` likewise.
- OTP lives in `src/use_case/member_liff.go`: `RequestMemberOTP` (rate-limit 3/hr),
  `VerifyMemberOTP` → returns `(memberExists, memberID)` and marks Redis pre-verified;
  **does not mint a session**. Actual OTP send/verify is a **separate shared messaging
  service** (`MESSAGING_SERVICE_BASE_URL`), not member-api. See [[reference_messaging_backend]].
- Company-without-LINE already solved: `company.go ResolveCompanyBySlug(slug)` →
  bola binding → company_id. OTP routes are already `/company/{slug}/members/otp|verify`.
- Member is **per-company** (`member.CompanyID` single field; `GetByPhone(companyID,phone)`),
  so one phone → up to one member row per company; the `{slug}` in the URL disambiguates.
  Cookie mint pattern to copy: `auth_v1.go:49-61` (Secure, HttpOnly, SameSite None/Lax,
  Expires=sess.ExpireAt, 204).

## Build decisions (user, 2026-09-07 — user is reporter/PO of both cards)
1. **Full OC-4348 now** (backend minter + web login FE + i18n + e2e), **front-running
   the OC-4345 spike** which is In Progress (another team, this week). User accepts
   rework risk — OC-4345 has NO decision doc yet, only input comments. Note the
   rework risk in an OC-4345 comment.
2. **Resolve company from member row** (not add company_id column): guard becomes
   `GetByID(sess.MemberID).CompanyID`. Safe because LineLogin resolves member via
   `GetByIdentAndCompany(inte.CompanyID,…)` ⇒ `member.CompanyID == inte.CompanyID`.
   Web-OTP session stores `integration_id = ""` (empty string satisfies NOT NULL varchar,
   guard ignores it now). No migration.
3. Contract: **reuse** `POST /v1/company/{slug}/members/otp` for OTP request; **new**
   `POST /v1/company/{slug}/auth/login` `{phone,otp_code}` → 204+cookie. Errors
   404 MEMBER_NOT_FOUND / 400 INVALID_OTP / 400 OTP_EXPIRED / 429 OTP_RATE_LIMITED.
   Refactor OTP-verify core into a shared helper so login+register use ONE mechanism
   (Rule 2). Login requires existing member — no auto-create (Rule 3).

Codegen is Go-based (edit `spec/v1.yaml` → `go run cmd/generate_fiber_interface/main.go`);
see [[reference_oc2plus_backoffice_codegen_is_go]]. Local 401 workaround for testing:
paste seeded cookie `oc2plus_crm_session=demo-customer-…` on :5183.
