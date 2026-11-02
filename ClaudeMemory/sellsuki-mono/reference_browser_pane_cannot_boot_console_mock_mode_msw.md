---
name: reference_browser_pane_cannot_boot_console_mock_mode_msw
description: "The built-in browser pane cannot register MSW's service worker, so the AI chat console in VITE_USE_MOCKS mode renders a blank page there; open/screenshot it with Playwright Chromium instead"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4a70053d-20ab-45c5-8ae9-5097398e8ddc
  modified: 2026-11-02T20:32:01.458Z
---

Seen 2026-11-03 (AI-374): `http://localhost:<port>/w/ws_demo_default/...` on a mock-mode Vite server stayed blank in the
in-app browser pane; console showed `[MSW] Failed to register the Service Worker ... An unknown error occurred when
fetching the script` although `/mockServiceWorker.js` served 200. main.tsx awaits the worker, so nothing renders.

**How to apply:** for "open the page before done" on the console in mock mode, run Playwright Chromium (the repo's
`@playwright/test`) against your own mock server on a non-5196 port (5196 is the e2e port every sibling session uses,
`reuseExistingServer: false`), e.g. an untracked `playwright.local-<card>.config.ts` that spreads the base config with
`baseURL` changed and `webServer: undefined`, and take `fullPage` screenshots. Delete the local config afterwards.
Related: [[reference_browser_pane_file_upload_via_datatransfer]].
