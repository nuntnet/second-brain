---
name: project-oc2plus-prod-v2-apikey-unverified
description: OC-4425 audit — production /v2/openapi/* on 3rdparty-api has NO API-key verification; oathkeeper rule that sets the headers exists on dev only
metadata:
  type: project
---

OC-4425 (WS-SEC2) audit, 2026-08-29. Request path to 3rdparty-api:
internet → `crmapi.oc2.plus` (ELB, public) → Ambassador Mapping →
`oathkeeper-proxy.share:4455` → Ory Oathkeeper (Maester Rule CRD) → service.

Config repos: `sre/configuration/api-gateway` (Ambassador Mapping + hostnames),
`sre/configuration/oc2plus` → `manifest/<env>/rule.yaml` (oathkeeper access
rules), `share/ory-helm` → `<env>/oathkeeper-values.yaml` (`managedAccessRules:
false` → CRD, not inline).

🔴 **On production**, one rule matches the whole domain
(`crmapi.oc2.plus<.*>`): authenticators `[bearer_token, anonymous]`, authorizer
`allow`, and it sets only `X-User-Id`/`X-User-Kind`/`X-Integration-Id`/`X-App-Id`
/`X-Session-Id` — **never `X-Api-Key-Id`/`X-Company-Id`/`X-Api-Scope`**. Prod runs
tag `v1.8.0`, which has `/v2/openapi/*` (incl. `POST /campaign/event/confirm` =
real point burn) but no `RequireApiKeyScope` middleware. So a client can send
those headers itself and pick any tenant, no API key. **Dev is fixed** — a second
rule `...-v2-openapi` verifies the key via `/v2/openapi/auth/whoami` and sets the
3 headers. Fix = copy that rule to `manifest/production/rule.yaml`; not built yet.

✅ `X-User-Id` can't be spoofed — oathkeeper mutator overwrites it from the
verified token on both crmapi and api.member. All 24 use cases taking an
`Identity` guard against anonymous (`X-User-Id:""`); regression test in 3rdparty!217.
Not yet done: fire the live proof at dev-th (Scope §2) — blocked on tsh session.
