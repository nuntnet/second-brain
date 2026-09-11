---
name: reference_fb_page_must_be_subscribed_to_app
description: Two independent gates decide whether a bound FB page receives messages — page→app subscription and app mode; the console's bind path had no subscribe at all until messaging !56
metadata:
  type: reference
---

An FB page that the admin console reports as **bound with `token_status: healthy`**
can still be completely silent. Two INDEPENDENT gates, often confused:

## Gate 1 — page → app subscription (`POST /{page-id}/subscribed_apps`)

Per page. Required **forever, for every customer page**. Publishing the app does
NOT remove it and does NOT do it for you.

It does **not** require the app owner to act: it is done programmatically with
the *customer's own page access token*, authorised by `pages_manage_metadata`,
which the console's OAuth already requests
(`apps/admin/src/lib/facebookSdk.ts` → `REQUIRED_SCOPES`).

## Gate 2 — app mode (Development vs Live)

Development: only people holding an App role (Admin/Developer/Tester) can
complete OAuth and have their messages delivered. Live (after App Review for
`pages_messaging` + `pages_manage_metadata`): any page admin can connect.

Publishing fixes ONLY gate 2. See [[project_fb_page_dev_mode_gate]].

## The bug that hid behind this (fixed: messaging-backend !56)

The console's path is **admin FE → chat-core
`/v1/workspaces/{id}/facebook-connection/connect` → messaging-backend LEGACY pair
`POST /v1/fb-apps/{app}/pages` + `.../exchange-token`**
(`chat-core/src/repository/messaging_client/facebook_connection.go`).

AI-125 added `SubscribePage` but wired it only into `ConnectPage`
(`POST /fb-apps/connection/connect`, the shared-app self-service flow) — which
the console never calls. So every page bound from the console was never
subscribed, went `healthy`, and received nothing.

!56 moved `SubscribePage` into `persistVerifiedPageToken` — the one function both
bind paths converge on — placed **before `PutPageToken`** so a page that cannot
be subscribed leaves no live credential in the secret store. New error
`ErrFBPageSubscribeFailed` (502 `fb_page_subscribe_failed`) and a
`page_subscribed` audit entry.

## Diagnostic signature (reusable)

`messaging.chat.channel_conversation` is written **on receipt, before** forwarding
to chat-core. So:

- **no row + no error = Facebook never delivered** → gate 1 or gate 2, not our bug
- row present but nothing in chat-core → our forwarding broke

Check order that worked: workspace exists in `chat_core` → `chat.fb_page_binding`
healthy → `chat.fb_app_audit_log` → `chat.channel_conversation` empty → curl the
public webhook URL (reachable ⇒ ingress fine) → then it is Meta's side.

Local webhook ingress is cloudflared: `ai-chat-local-api.bearyweb.com` →
`localhost:8100`, in `~/.cloudflared/config.yml` (tunnel `bola-local`).

Still open: a page **unsubscribed later** on Meta's side still reports healthy —
`RunDailyHealthCheck` verifies the token, not the subscription.
