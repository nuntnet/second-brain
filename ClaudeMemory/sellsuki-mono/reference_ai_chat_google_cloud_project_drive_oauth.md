---
name: reference_ai_chat_google_cloud_project_drive_oauth
description: "AI chat Drive import's Google Cloud project is ai-chatsystem-510115 (number 739355502757), reachable only as the sellsuki account (authuser=1); two Picker keys, one GitLab var"
metadata:
  node_type: memory
  type: reference
  originSessionId: b32c7a82-501f-4f85-932d-42d351fc7895
  modified: 2026-11-26T07:24:49.266Z
---

Google Drive KB import (console + knowledge-connector) uses Google Cloud project **AI-chatsystem**, id `ai-chatsystem-510115`, number `739355502757` (= `VITE_GOOGLE_APP_ID`).

- In the in-app browser the default Google account (authuser=0) is a personal one that cannot see it; open URLs with `&authuser=1&project=ai-chatsystem-510115` (`?project=739355502757` alone gives "You need additional access").
- OAuth client `ai-chat-oauth` (739355502757-ka24…) holds origins + redirect URIs for local/staging/prod.
- Two Picker API keys: `ai-chat-kb-picker-local-staging` (referrers local + staging) and `ai-chat-kb-picker-production` (`https://chat.sellsuki.com/*`). As of 2026-11-26 GitLab console var `VITE_GOOGLE_PICKER_API_KEY` had only scope `*` = the staging key; production needs the same name with environment scope `production` (build_code_production has `environment: production` via SRE template `.env_production`, so the scoped var wins).
- Consent screen was still "Testing" (one test user) on 2026-11-26; config includes restricted scope `drive.readonly` (folder mode) → unverified warning + 100-user cap if published without verification.
- Compare key values by last 4 chars only; never print them.

Related: [[reference_staging_ai_chat_console_drive_recipe]]
