---
name: project_meta_app_sellsuki_a_prod_state_2026_11_14
description: "Prod Meta app \"Sellsuki - A\" (323033617839137) observed 2026-11-14 — Development mode, feed unsubscribed, webhook fields at v27.0 vs Graph calls v25.0, login config 1459004999493010 = 3 pages_* perms, all Standard access"
metadata:
  node_type: memory
  type: project
  originSessionId: 17b79f90-4e6f-4d55-85bf-48f14625212a
  modified: 2026-11-23T05:32:48.498Z
---

Read-only check in Meta for Developers (nunt signed in in the in-app browser), 2026-11-14, for AI-442 (FB comment → private reply):
- App mode Development; App Review not submitted; every `pages_*` permission at Standard access.
- Webhooks → Page: `messages`, `messaging_postbacks` Subscribed; `message_echoes`, `messaging_referrals` Unsubscribed. **`feed` Subscribed at v27.0 — seen 2026-11-23** (was Unsubscribed on 11-14; who turned it on is unknown). Field webhook version **v27.0**, while messaging's `CHAT_FB_GRAPH_API_VERSION` is v25.0. Note: the Webhooks page opens on the **User** object, whose field list also has a `feed` row (Unsubscribed) — switch to **Page** in the left list before reading.
- Login for Business config "AI Chat production" `1459004999493010` (user access token): `pages_messaging`, `pages_show_list`, `pages_manage_metadata`, since 2026-11-20 ~21:30 `pages_read_engagement` (added for AI-496 / console !187, verified by reopening the wizard), and since **2026-11-23 ~12:25 `pages_read_user_content`** (added on nunt's word for the AI-442 prod private-reply test; wizard showed 5 options → Save returned to the config list). A Page connected before that must be reconnected to grant it. Still Development mode + Standard access, so only app-role users can grant it until App Review.
- Reading the permissions means opening the config's Edit wizard → Permissions step → typing in the selector's search box; leave with Cancel.

**Why:** the next person preparing App Review or a webhook change needs the baseline without re-clicking. **How to apply:** re-check before acting, because this state changes as soon as P1–P4 in docs/cards/AI-442.md §8 are done.
