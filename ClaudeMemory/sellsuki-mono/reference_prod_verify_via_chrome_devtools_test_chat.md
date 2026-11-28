---
name: reference_prod_verify_via_chrome_devtools_test_chat
description: Verifying AI chat cards on production — chrome-devtools MCP browser nunt signs into; fetch chat-core.sellsuki.com from the page; Test_Chat-only writes; what prod Test_Chat cannot do
metadata:
  node_type: memory
  type: reference
  originSessionId: e4523e63-a588-45fc-b5bb-8196edcf4ad4
  modified: 2026-11-28T13:21:11.176Z
---

Used 2026-11-28 when staging-th was down.

- `chrome-devtools` MCP (own profile `~/.cache/chrome-devtools-mcp/chrome-profile`); nunt signs in once at https://chat.sellsuki.com (never type the password). One agent drives it at a time.
- Console API = `https://chat-core.sellsuki.com`, cookie session. Read it with `fetch(..., {credentials:'include'})` inside `evaluate_script`. **Never `get_network_request`**, which prints the session cookie.
- nunt allowed writes on prod **only in Test_Chat** (company `612b5454…`, workspace `2fde1c3e…`, Page "Test ChatBot"). nunt is also company_admin of real companies; never open those.
- Prod limits seen in Test_Chat: try-chat ("chat as a customer") flag off; file-service upload 403 `attachment_storage_denied`; dashboard 503 `dashboard_not_configured`; the only room is a real Facebook thread (never send in it); ad-spend entries cannot be cleared from the UI.
- Brief template: monorepo `docs/review-packs/verify-2026-11-28/BRIEF-PROD.md`.
