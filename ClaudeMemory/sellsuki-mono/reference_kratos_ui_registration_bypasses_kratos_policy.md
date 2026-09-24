---
name: kratos-ui-registration-bypasses-kratos-policy
description: kratos-ui-go signup creates identities via Kratos ADMIN CreateIdentity — no password policy, no CSRF, no rate limit; proven locally with password "123" and no csrf_token (PAT-2730 area)
metadata:
  type: reference
---

`SubmitRegistration` (app/controllers/registration_controller.go, same on develop) does NOT submit the Kratos self-service registration flow for password signup — it calls `AdminClient.IdentityAPI.CreateIdentity`. Consequences, verified on local 2026-09-25:

- Password `123` accepted (Kratos min-8 / breach check never runs; UI has no validation either).
- POST without `csrf_token` accepted.
- OIDC signup also finishes through admin CreateIdentity (schema `refid`), so Kratos's native duplicate-account linking never runs; duplicate email → 409 → redirect to login, Google data lost, and the typed email is not tied to the Google email (email squatting possible).
- `utils.GenerateHashPW` uses an all-zero salt (`make([]byte,16)` never filled). Currently harmless: Kratos v25 hashes the plaintext `password` and ignores `hashed_password` (live salts are random) — dead-but-dangerous.

Kratos itself: new code invalidates the previous one (4070006); flow expiry (20m) is not extended by resend; 5 wrong codes kill the flow (410 self_service_flow_replaced). Resend is unthrottled (3 mails/min observed). Deployed Kratos config/ingress rate limits live outside this repo — unverified.
