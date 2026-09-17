---
name: reference_fb_page_delivery_diagnosis_endpoint
description: messaging-backend can ask Meta directly why a page is silent, using the page token it already holds — GET /v1/fb-apps/{app}/pages/{page}/delivery-diagnosis; it proved เป็ดน้อย's Meta config is identical to a page that works
metadata:
  type: reference
---

`GET /v1/fb-apps/{appId}/pages/{pageId}/delivery-diagnosis` (messaging-backend,
service-token) asks the **Graph API** what Meta believes about a bound page,
using the Page Access Token already in the secret store. Read-only.

It exists because the two remaining causes of a silent page are invisible to
every local status column — the app subscribed without the `messages` field,
and another app holding thread control via the Handover Protocol — and seeing
either otherwise needs the **page owner's own Facebook settings**, which a
platform operator does not have for a customer's page.

## Calling it locally

```bash
curl -s -H "X-Service-Token: local-dev-token" \
     -H "X-User-Id: <actor>" -H "X-Company-Id: <binding's company_id>" \
     "http://localhost:8100/v1/fb-apps/<appId>/pages/<pageId>/delivery-diagnosis"
```

`X-Company-Id` must be the binding's own `company_id` (it differs per page —
some rows carry the placeholder `company-fb-pilot`, others a real UUID), or the
ownership mask answers `fb_page_not_bound`.

## What it found (2026-09-18, the เป็ดน้อย hunt)

| page | subscribed apps | delivers |
|---|---|---|
| ร้านขนมบ้าน'เป็ดน้อย. | 1 — ours, `messages`+`messaging_postbacks` | ❌ |
| Sellsuki Chatbot | 1 — ours, `messages`+`messaging_postbacks` | ✅ |
| Marketing Space | 1 — ours, + `messaging_checkout_updates` | ✅ |

**เป็ดน้อย's Meta-side configuration is identical to a page that works.** That
kills the whole Meta-config family of hypotheses at once — subscription,
subscribed fields, and a competing app (only ONE app is subscribed to any of
these pages, so nothing is holding thread control).

`secondary_receivers` returns Graph error code=100 for **every** page,
including the working ones, so it is an endpoint/permission limit rather than
a per-page finding. The response reports that error instead of an empty list
on purpose: "could not look" must never render as "there is nothing there".

## Trap

Use `GetPageToken`, not `Get`, for a page token — they read different Vault
fields, and the wrong one answers `vault secret missing "app_secret" field`,
which reads as a broken secret store rather than a mistyped accessor.
