---
name: reference_staging_fb_webhook_404_is_page_unbound_silently
description: "messaging-backend answers FB webhook 404 (fb_page_not_bound) for a page someone unbound, with NO log line — messages to that page vanish. Diagnose from chat.fb_app_audit_log + fb_page_binding in staging_sellsuki_messaging. Seen 2026-10-28 (Narawit unbound Test ChatBot)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2b0f396b-5941-4a79-8e15-d9e0597e4520
  modified: 2026-10-28T09:44:09.064Z
---

2026-10-28 triage of "Udomdej messaged the page, nothing arrived" (staging, app `1722666062152698`):

- `request_log` for `/webhook/fb/:appId` showed 65× **404** from 07:40Z. The webhook route maps 404 to `ErrFBAppNotFound` / `ErrFBPageNotBound` (`src/interface/fiber_server/route/fb_webhook/errors.go:22,32`) and logs **nothing** for it; the response body is empty in the log.
- `chat.fb_app_audit_log` (db `staging_sellsuki_messaging` on pg18) explained it in one query: `page_unbound` for page `1366041813265036` at 07:34:41Z by `c5829c9e` = narawit.pan@sellsuki.com, who had bound it to workspace "Test ChatBot" (`618acd1f`, company `018a163e` "Test ChatBot") on 10-26.
- Only `Sellsuki Chatbot` `1263419410196283` → AI Chat Pilot `5266e73a` remained bound.
- Same day, 07:10Z: 24× **500** `failed to fetch attachment from facebook cdn — host is not an allowed facebook cdn host` (allowlist `*.fbcdn.net, *.fbsbx.com, *.cdninstagram.com`, env `CHAT_FB_MEDIA_ALLOWED_HOSTS`; the rejected host is not logged). Meta retried, then the retries were dropped as `timestamp_window_exceeded`, so that message was lost too.
- After 08:14Z nothing at all arrived from Meta.

**How to apply:** for "message to the staging page didn't arrive", run in this order:
1. Group the webhook `request_log` lines by status.
2. A 404 means look at `fb_app_audit_log` for `page_unbound`.
3. Zero requests means Meta did not deliver. Check whether the sender holds an app role (test app, [[reference_staging_fb_app_is_a_test_app]]).

Messaging tables live in schema `chat` (`channel_conversation`, `fb_app`, `fb_app_audit_log`, `fb_page_binding`, `send_receipt`). Related: [[reference_fb_page_must_be_subscribed_to_app]], [[reference_webhook_ack200_and_drop_looks_like_no_delivery]].
