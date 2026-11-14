---
name: project_meta_app_sellsuki_a_prod_state_2026_11_14
description: "Prod Meta app \"Sellsuki - A\" (323033617839137) observed 2026-11-14 — Development mode, feed unsubscribed, webhook fields at v27.0 vs Graph calls v25.0, login config 1459004999493010 = 3 pages_* perms, all Standard access"
metadata:
  node_type: memory
  type: project
  originSessionId: 17b79f90-4e6f-4d55-85bf-48f14625212a
  modified: 2026-11-14T06:43:51.407Z
---

Read-only check in Meta for Developers (nunt signed in in the in-app browser), 2026-11-14, for AI-442 (FB comment → private reply):
- App mode Development; App Review not submitted; every `pages_*` permission at Standard access.
- Webhooks → Page: `messages`, `messaging_postbacks` Subscribed; `feed`, `message_echoes`, `messaging_referrals` Unsubscribed. Field webhook version **v27.0**, while messaging's `CHAT_FB_GRAPH_API_VERSION` is v25.0.
- Login for Business config "AI Chat production" `1459004999493010` (user access token): `pages_messaging`, `pages_show_list`, `pages_manage_metadata`. No `pages_read_user_content`.
- Reading the permissions means opening the config's Edit wizard → Permissions step → typing in the selector's search box; leave with Cancel.

**Why:** the next person preparing App Review or a webhook change needs the baseline without re-clicking. **How to apply:** re-check before acting, because this state changes as soon as P1–P4 in docs/cards/AI-442.md §8 are done.
