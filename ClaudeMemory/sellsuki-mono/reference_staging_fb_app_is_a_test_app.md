---
name: reference_staging_fb_app_is_a_test_app
description: "AI chat staging's Facebook app 1722666062152698 ('Sellsuki AI Chat Staging') is a TEST app of Sellsuki-New, so it can never go Live; in Development mode only app-role users can log in or (for Messenger) be served. Domains for ai-chat-admin.staging.sellsuki.com are configured as of 2026-10-12"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-28T09:58:25.745Z
---

Checked in the Meta App Dashboard on 2026-10-12. The PO's Chrome is "Browser 1" in the Claude extension and is logged in to Meta. "Browser 2" is not.

- App `1722666062152698` ("Sellsuki AI Chat Staging") shows "You are currently editing a test version of Sellsuki-New". The App Mode toggle tooltip says "Test apps can't be switched to live mode".
- Already configured:
  - App domains include `ai-chat-admin.staging.sellsuki.com`, `ai-chat-admin.staging-th.sellsuki.com` and `chat.sellsuki.local`.
  - Login with the JavaScript SDK is Yes, and Allowed JS SDK domains list the same three.
  - The Valid OAuth Redirect URI is `https://ai-chat-admin.staging.sellsuki.com/`.
- App roles (10-12): the only Administrators were Nati Namvong and "Nunt- Janwaowam" (the PO), with no testers. **As of 2026-10-28:** Admins are Nati, Best Narawit and Nunt; Testers are Nattarida Panyo and Udomdej Chuenjit (Udomdej added 10-28 ~16:55 ICT).
- 🔴 **The test app's roles are separate from the parent Sellsuki-New `235756003289739`.** On 10-28 Udomdej was a Tester on the parent, not on the test app, so every message he sent to Sellsuki Chatbot stayed in the page inbox and no webhook arrived. Within a minute of being added on `1722666062152698` his messages flowed: 5× webhook 200, a new PSID row, and AI replies. Always open `/apps/1722666062152698/roles/roles/`, not the parent's page.
- The page shows "Facebook Login for Business requires advanced access (public_profile)". That matters only beyond role users.

**How to apply:**
- "JSSDK unknown host domain" or "Can't load URL" on connect means a domain is missing in the two lists above.
- Anyone who must connect a page, or must message the page and be answered, has to be added under App roles, for example as a Tester. Development mode serves only role holders.
- Real, unknown customers need the live parent app, a product decision.

Related: [[reference_fb_page_must_be_subscribed_to_app]].
