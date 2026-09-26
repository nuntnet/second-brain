---
name: member-api-dev-otp-hits-staging-messaging
description: OC2Plus member-api on DEV calls the STAGING messaging-backend (values-development MESSAGING_SERVICE_BASE_URL = …svc.sellsuki, not .sellsuki-dev) — dev shops configured in CCS3-dev get OTP 503 action_not_configured
metadata:
  type: reference
---

`backend/oc2plus-line-crm-service-member-api/deployment/values-development.yml` sets
`MESSAGING_SERVICE_BASE_URL: http://sellsuki-messaging-backend-svc.sellsuki` — the **staging**
namespace. Unchanged since 2024-02-28 (predates a dev messaging-backend). Same value on
origin/develop and origin/main as of 2026-09-26. Every other dev consumer uses `sellsuki-dev`
(3rdparty-api since 2024-11, CCS `…sellsuki-dev:80`).

Effect: a shop sets up its OTP provider through CCS3 on dev → rows land in **dev** messaging
(`is_routable: true`), but member-app OTP goes to **staging** messaging → `message_action` miss →
404 `action_not_configured` → member-api maps to 503 `OTP_UNAVAILABLE`. Dev OTP only "works" for
shops that also happen to be seeded on staging (and then uses staging's SMS provider).

How it was proven (2026-09-26, shop slug `dajr1ogrfm5d2pi0b5p0`, company `fe621d66-0255-4647-b1de-6ae081b6864f`):
member-api pod log `messaging_repository.RequestOTP` 404 at 05:44:56Z → the matching
`message_action_repository … item not found` + `POST /v1/otp/request` appeared in the **`sellsuki`**
namespace pod (version 66a094bd), NOT in `sellsuki-dev` (22bf972c), whose logs only show that
company's `GET /v1/messaging/config` calls.

Lesson: when an OTP/messaging miss looks like missing tenant data, first check WHICH environment
the caller dials — read the deployed env (`kubectl get deploy -o json`), then find the request in
the callee's logs. See [[reference_messaging_otp_needs_a_message_action_row]] (same symptom, different cause).
