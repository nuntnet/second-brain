---
name: reference_messaging_otp_needs_a_message_action_row
description: "messaging-backend OTP needs TWO rows per tenant — provider config (CCS3 UI creates it) and a message_action row that NOTHING creates; the action miss lies and reports error_code provider_not_configured"
metadata:
  type: reference
---

`RequestOneTimePassword` ([src/use_case/one_time_password.go:40](backend/sellsuki-messaging-backend/src/use_case/one_time_password.go:40))
does **two** per-tenant lookups before it ever touches the SMS vendor:

1. `message_action` by `(owner_id, owner_type, service_type='otp', code)` — the
   action code the caller sends in `X-Action-Code` (OC2Plus member-api sends
   `oc2plus.crm.otp`, hardcoded in `messaging_repository.setHeaders`).
2. `message_provider_config` by the `provider_config_id` that action points at.

🔴 **Both misses return `model.ErrProviderConfigNotFound` → 404
`error_code: "provider_not_configured"`.** So that error code does NOT mean the
provider config is missing — most often the config is fine and the ACTION row
does not exist. `action_not_configured` exists in `helper/errors.go` but the OTP
path never uses it. Consumers then map the 404 to their own opaque error
(member-api → 503 `OTP_UNAVAILABLE`), so the client sees nothing useful at all.

**Nothing in the platform creates the `message_action` row.**
`MessageActionConfigRepository` is `HealthCheck` + `GetByCodeAndID` only
(`src/use_case/repository/repository.go:66`); the HTTP surface is
`/v1/messaging/config` + `/v1/messaging/providers` and has no action endpoint;
no other service in the workspace mentions the table; the repo's own README
(§"Adding a New Action") tells you to `INSERT` it by hand. The CCS3 messaging
settings screen creates only half the setup and looks finished.

It is **not** a missing migration — the table and all 11 columns exist. Proof
without DB access: the repository returns `ErrDatabase`/500 when the query
itself fails and `ErrNotFound`/404 only when `len(results) == 0`
(`message_action_repository/postgresql.go:86-127`), and we got the latter.
This is tenant DATA with no creation path — same family as a permission that
never reached a role preset, not [[reference_migration_files_are_not_applied_schema]].

**How to diagnose in 3 minutes (2026-09-22, staging, OTP 503 on member app):**
- member-api pod logs → `messaging_repository.RequestOTP` logs the upstream
  status + body verbatim, giving the real `error_code`.
- messaging-backend pod logs at the same timestamp → the `"item not found"`
  line names the repository that missed (`message_action_repository` vs
  `message_provider_config_repository`) — that is what splits the two causes.
- read-only check of what IS configured, no DB creds needed:
  `kubectl port-forward -n sellsuki svc/sellsuki-messaging-backend-svc 18099:80`
  then `GET /v1/messaging/config` with `X-User-Id: <company uuid>`,
  `X-User-Kind: sellsuki.company`. Returns configs + per-service-type quota.
- Fixing the vendor credentials changes nothing while the action row is absent —
  the credential is read at step 2, after the miss. Verified: credentials were
  updated at 05:30:52Z and the 05:32 failures were byte-identical.

Staging on 2026-09-22: two companies had called `/v1/otp/request`
(`f2bf928c-7cd0-4059-9326-82fddeba4933`, `c01b0bdb-61e9-4443-8c86-8d8b63123811`),
404 on every call, zero successes.

⚠️ From that I concluded "the action table has no OC2Plus rows at all there" —
**wrong**. The table held 19 companies with `oc2plus.crm.otp` already; the two
failing ones were simply never seeded. Zero successes in a log window is a
statement about who called, not about what the table holds. Count the table.

Second company's failure had a different cause again: `c01b0bdb…` has no
provider config at all, so no backfill can help it — that tenant has to
configure a provider first.

**Fixed 2026-09-22, three MRs, merge in this order** (producer → proxy →
consumer; all use a three-state `*bool` so merging out of order degrades rather
than breaks):
1. messaging-backend !64 — `ensureDefaultActions` creates the route when a
   provider config is created, `model.DefaultActionCodes` is the registry,
   action repo gains `Create`, OTP path returns `action_not_configured`,
   `scripts/sql/backfill-otp-actions.sql`, AGENTS.md/README table names fixed.
2. CCS !336 — passes `is_routable` through (it decodes into a struct, so an
   unlisted field is dropped in transit).
3. backoffice-api !615 — readiness row reports `partially_configured` +
   `detail.missing = otp_routing` instead of ready.

Backfill already run: **staging 6 rows, development 2, production NOT touched**.
Ledger: `docs/cards/2026-09-22-otp-action-routing.md`.

Still open: `SetPrimary` does not repoint routes, so "set as primary" changes the
badge without moving traffic — the same family of lie, deliberately left for its
own card.

See [[reference_messaging_backend]] · [[reference_oc2plus_otp_session_fails_3rdparty_consent]]

**The readiness gate does not catch it.** backoffice-api's `checkOTPProvider`
(`src/use_case/company_readiness.go`) calls `HasOTPConfig` → GET
`/v1/messaging/config`, i.e. it verifies the provider config ONLY. A company
with a config and no action row is reported ✅ ready while every OTP request
502/503s. Any fix should make that check assert the action link too.
