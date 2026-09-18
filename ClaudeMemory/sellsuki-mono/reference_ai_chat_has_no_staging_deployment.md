---
name: reference_ai_chat_has_no_staging_deployment
description: The AI Chat path does not exist on staging — chat-core's deploy job has never been run and messaging-backend's chat module is deliberately disabled there, so the AI-121 e2e release gate cannot pass and never has run
metadata:
  type: reference
---

Checked 2026-09-18 against real staging.

| service | state on staging |
|---|---|
| **chat-core** | **not deployed.** `deploy_staging_th_arm` is `when: manual` and has never been run. Public `chat-core.staging-th.sellsuki.com` → **503** (ingress exists, no healthy backend); internal `sellsuki-chat-core-svc…` → 404 |
| **messaging-backend** | pod healthy — `/system/liveliness`, `/metrics`, `/v1/messaging/config` all 200 — but **`CHAT_MODULE_ENABLED: "false"`** in `deployment/values-staging.yml`, so `/webhook/fb/{app}` and `/v1/fb-apps/*` are **404** |

The flag is off *deliberately and for a stated reason* (values-staging.yml, its own comment): the chat module's boot health-check calls chat-core's `/system/liveness` and **fatals the whole pod** when it fails, which would block the messaging deploy entirely — including the OTP/SMS/email side that other products depend on. Flip it back to `"true"` only once chat-core (plus Vault/Kafka/Redis values) is actually provisioned there.

So the order is: deploy chat-core → provision its deps → flip the flag → then the webhook path exists.

## What this means for the AI-121 e2e suite

`testing/ai-chat-e2e-playwright`'s staging project **cannot pass**: scenario 1
posts a signed webhook at a route that 404s. It also has never been run —
no `staging-smoke` job appears in any recent pipeline, only `verify` and
`verify-realtime-observer`.

Its fixtures were never created either. The project has exactly **2** CI
variables, both plain URLs:

```
E2E_CHAT_CORE_BASE_URL  https://chat-core.staging-th.sellsuki.com
E2E_MESSAGING_BASE_URL  http://sellsuki-messaging-backend.sellsuki.internal.staging-th.sellsuki.com
```

`E2E_FIXTURE_JSON`, `E2E_OPERATOR_COOKIE`, `E2E_SERVICE_TOKEN`,
`E2E_META_MOCK_BASE_URL`, `E2E_OPENROUTER_MOCK_BASE_URL` and
`E2E_EVENT_SINK_BASE_URL` do not exist at all — so the outer mocks and the
event sink the suite reads may never have been stood up.

## Useful: the cluster is reachable from this machine

`sellsuki-messaging-backend.sellsuki.internal.staging-th.sellsuki.com`
resolves to `10.21.3.172` and answers over HTTP from the laptop. Once the
services are deployed, the staging suite can be driven from here without CI —
`E2E_MESSAGING_BASE_URL` is that internal host, not a public one.

Related: [[reference_procfile_messaging_disables_chat_module]] — the same flag
caused the same 404 locally and cost a day.
