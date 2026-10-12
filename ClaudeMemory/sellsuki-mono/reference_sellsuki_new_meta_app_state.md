---
name: reference_sellsuki_new_meta_app_state
description: "Meta app Sellsuki-New 235756003289739 (business 263014493891012) is Live but pages_messaging is only Standard access (no App Review), so it still answers only app-role users; its Page webhook pointed at a local tunnel ai-chat-local-api.bearyweb.com on 2026-10-12. One callback URL per app; messaging-backend drops pages bound elsewhere, so staging+prod sharing one app needs a relay"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-12T07:54:22.292Z
---

Checked read-only in the PO's Chrome (Browser 1) on 2026-10-12, when the PO asked to use Sellsuki-New for staging and production so the public, not just testers, can chat.

- **App mode:** Live. App type is Business, owned by business `263014493891012`.
- **`pages_messaging`:** Standard access, "No App Review requested" (347 calls). The same goes for `pages_manage_metadata`, `pages_show_list`, `pages_read_engagement` and `business_management`. `public_profile` advanced access says "Verification required".
  - Standard access serves only people with a role on the app or its business.
  - **Live mode alone does not let the public message the page.** Advanced Access through App Review is required, and Business Verification is likely too.
- **Webhooks → Page callback:** `https://ai-chat-local-api.bearyweb.com/webhook/fb/235756003289739`, someone's local tunnel. Repointing it cuts that person off; ask who owns it first.
- **The staging test app** `1722666062152698` is a test version of Sellsuki-New. See [[reference_staging_fb_app_is_a_test_app]].

**Our side:**
- messaging-backend routes webhooks at `/webhook/fb/:appId`.
- `ResolveRouting` returns `ErrFBPageNotBound` for a page not bound in its own DB (`src/chat/use_case/fb_app_use_case.go` ~505-528). So two environments cannot share one app without a page_id → environment relay, since Meta allows one callback URL per app.
- Staging selects the app through `CHAT_FB_SHARED_APP_ID` / `CHAT_FB_SHARED_APP_OWNER_COMPANY_ID` in `deployment/values-staging.yml`. The app row is registered via `POST /fb-apps`.
- Production and development values have no FB config.

**How to apply:**
- Before promising "public users can chat", check the App Review status of `pages_messaging`.
- Staging is off from 21:00 to 09:00, so real customers on a staging-bound page get no answer at night.
