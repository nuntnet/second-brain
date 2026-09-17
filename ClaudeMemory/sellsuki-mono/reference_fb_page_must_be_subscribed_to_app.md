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

**Decided by the user (2026-09-17):** the fix is NOT to fold the subscription
into the existing token health. Token and subscription become **two separate
statuses on screen** — a page can hold a valid token and still be unsubscribed,
and the admin must be able to tell those apart. `RunDailyHealthCheck` gets an
`IsPageSubscribed` probe alongside the token check.

## Proven asymmetry (2026-09-17, after every layer on our side was fixed)

Two pages, **one app** (`235756003289739`), one callback, one cloudflared
tunnel, three minutes apart:

| page | subscribed | result |
|---|---|---|
| marketing space `268502970215081` | yes (bound by hand in early dev) | ✅ webhook delivered, all hops, DB row |
| ร้านขนมบ้าน'เป็ดน้อย. `1475633739322428` | yes — `page_subscribed` written 21:28:14 | ❌ nothing, not even hop `receive` |

Everything on our side was proven live: public tunnel answers from the
internet, route serves, app active with verify_token + secret, token
`healthy`, token identity verified (`ResolveTokenPageID` matched, no
`page_token_exchange_page_mismatch` audit), zero `chat_webhook_rejected_total`.

So a page can be subscribed, healthy, and still silent. The remaining
page-level explanation not disprovable from our side is the **Handover
Protocol**: if another app is the page's Primary Receiver, Meta sends
`standby` events, and we subscribe only `messages,messaging_postbacks` — so we
receive nothing rather than receiving and rejecting. Checking it needs the page
token against the Graph API, which we do not hold.

### Ruled out on 2026-09-17 night, with evidence

- **Page→app subscription** — Meta's OWN console lists เป็ดน้อย with
  `messages, messaging_postbacks` (Messenger API Settings → Generate access
  tokens). Not just our API's word for it.
- **App Mode** — flipped the app to **Live** (Site URL moved off the invalid
  `chat.sellsuki.local` to `ai-chat-admin.staging-th.sellsuki.com`, app domain
  added, `chat.sellsuki.local` deliberately kept so local OAuth survives).
  Messaged the page again: still zero. **Development mode was not the gate.**
- **App-level config** — one callback URL for all pages
  (`ai-chat-local-api.bearyweb.com/webhook/fb/{app}`), verified answering from
  the public internet after every restart. App roles: the user is an app
  Administrator; Testers 0.
- **Our ingress** — `chat_hop_duration_seconds{hop="receive"}` never increments
  for this page. Nothing arrives to reject.

What is still unseen: เป็ดน้อย's own Page settings. The user's Business
Managers (Ch.Erawan, Fuse Sellsuki, Genelab, MACfig, Sellsuki) **do not
contain that page** — it is a customer page connected by OAuth only — so
Handover Protocol / connected apps cannot be inspected without the page
admin. That asymmetry (own page delivers, external page does not) is the only
structural difference left.
