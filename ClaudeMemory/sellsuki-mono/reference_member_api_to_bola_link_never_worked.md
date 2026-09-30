---
name: reference_member_api_to_bola_link_never_worked
description: "member-api → BOLA was broken three independent ways on every env (base URL unset, /contacts/link missing /v1, X-Api-Key vs X-System-Token) — so LIFF registration never created a BOLA follower and LINE login could not fetch oa-info; BOLA_API_KEY must equal BOLA's SYSTEM_ADMIN_TOKEN and only a human can set it"
metadata:
  type: reference
---

Found 2026-09-30 on dev (OC-4523 follow-up; fixed in member-api !174, bola-backend !208, member-api !173):

1. `BOLA_SERVICE_BASE_URL` was in **no** values file (dev/staging/prod) → default `http://localhost:8080`, the pod called itself. Boot log said so all along: `config.service_url_unusable … points at loopback`. **Read that log line first** when member-api ↔ BOLA misbehaves.
2. `LinkContact` posted `/contacts/link`; BOLA serves `/v1/contacts/link` (measured on dev: 404 vs 401).
3. `LinkContact` sent `X-Api-Key`; BOLA's `MachineOrAdminGuard` reads only `X-System-Token`, so a right call still fell to the admin-session path (401).

Symptoms that looked unrelated: BOLA Contacts = 0 after a member registered, OC2Plus contact list with no LINE display name, and (via `GetLiffOAInfo`, same base URL) LINE id_token login always falling back to OTP.

**Human step nobody can skip:** member-api's `BOLA_API_KEY` must equal BOLA's `SYSTEM_ADMIN_TOKEN`; put it in `oc2plus-crm-secret`. It was not set on the dev pod (env list checked). Do NOT add a `secretKeyRef` to values before the key exists — the pod will not start. Staging/production values still lack `BOLA_SERVICE_BASE_URL`.

Related: [[reference_member_api_service_urls_were_never_set]] (same class: THIRDPARTY/FILE_SERVICE), [[reference_envdefault_localhost_masks_missing_config]], [[project_oc4523_line_login_cards]].
