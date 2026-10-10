---
name: reference-member-app-local-test-mode-not-active
description: "2026-10-11 live drive of the OC2Plus member app on local — launch config web-member is a MOCK (VITE_USE_MOCK_MEMBERSHIP), the real-API one is web-member-oc3126; company localtest IS open to members; OTP in test mode failed 502 UPSTREAM_UNAVAILABLE because member-api's TEST_KEY did not match the key the proxy sends"
metadata:
  node_type: memory
  type: reference
  originSessionId: 5077123b-d938-4a36-b54b-335561e7b9a9
  modified: 2026-10-10T18:04:41.604Z
---

Observed 2026-10-11 ~00:55 while driving `.claude/verify/oc2plus-member/features/register-login.md`:

- `.claude/launch.json` **`web-member` and `web-member-qa` run the member app with
  `VITE_USE_MOCK_MEMBERSHIP=true`** — a mock membership, not member-api. The config
  that proxies to the real API on :5183 is **`web-member-oc3126`**, which sets
  `VITE_LOCAL_DEV_TEST_SECRET=local-qa-test-key-oc4348`.
- `GET http://localhost:8102/v1/company/localtest/availability` → `{"available":true}`:
  the `oc2plus_company_activation` row for `localtest` exists (the inventory had
  flagged it as probably missing).
- `/localtest/login` renders; `POST /v1/company/localtest/members/otp` (phone
  `0899999999`, header injected by the Vite proxy) → **502 `UPSTREAM_UNAVAILABLE`**
  from member-api itself (`otp upstream unavailable`, issue `7f4b15a4…`). That means
  the request went to the real OTP upstream: test mode did not engage, so the running
  member-api's `TEST_KEY` ≠ `local-qa-test-key-oc4348`, or `ENVIRONMENT` is not one
  of development|staging|local|test. Agents cannot read `.env` to confirm which.
- The Claude sandbox blocks HTTP to localhost (curl exit 000) and the overmind socket;
  probe with `dangerouslyDisableSandbox` or the preview browser.

**How to apply:** before driving the member app, ask the user (or check outside the
sandbox) that member-api's `TEST_KEY` equals the frontend secret; otherwise every
recipe past the OTP step is `verified-unreachable`. Use `web-member-oc3126`, never
`web-member`, for a real-API drive. Each OTP attempt spends one of 3 per phone per
hour. See [[project-pstack-as-workflow-benchmark]], [[reference_oc2plus_member_api_test_mode_login]].
