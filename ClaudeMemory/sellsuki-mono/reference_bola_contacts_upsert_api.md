---
name: reference-bola-contacts-upsert-api
description: "How to test BOLA's POST /v1/contacts/upsert against staging (auth, field mapping, throughput)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: b5634410-3d38-4fdf-80e0-5b4a32f2c004
---

`POST /v1/contacts/upsert` on bola-backend ([route.go:604](backend/bola-backend/src/interface/fiber_server/route/v1/customer/route.go:604)) is async — returns `202 {job_id}`, poll via `GET /v1/contacts/import-jobs/{job_id}?workspace_id=...`.

**Auth for staging tests**: route uses `FlatAdminGuard`, which reads `workspace_id` from the **query string** (not path) and authenticates via `Cookie` header (`sellsuki_session_staging` etc., same cookie as logging into the sellsuki platform in a browser) — no separate BOLA token needed. Swagger UI's `workspace_id=...` query param in the URL is this same param, not just doc styling.

**Field mapping**: schema only has `phone`, `line_uid`, `line_oa_id`, `first_name`, `last_name`, `email`, `note`, `tags`, `custom_fields`. Any source columns without a matching field (e.g. a CRM export's `account_id`, `campaign_id`, loyalty fields) go into `custom_fields` (RFC-7396 merge patch).

**Phone validation**: backend validates E.164 (`^\+[1-9]\d{6,14}$}`, [lon.go:24](backend/bola-backend/src/use_case/lon.go:24)). Thai 10-digit `0xxxxxxxxx` numbers need `options.normalize_phone: "+66"` in the request to auto-convert — raw `0` numbers fail validation otherwise.

**Throughput**: upsert processes roughly ~14 contacts/sec per job (observed: 5,000-contact batch took several minutes; a 411-contact batch took ~29s). For large imports (tested with 75,411 rows), split into ~5,000-record batches and expect each batch to take 5-6 minutes to reach `status: completed`.

Related: [[project_bola_workspace_scoping_bugs]], [[project_bola_kratos_sso_staging]]
