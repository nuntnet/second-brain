---
name: reference_console_e2e_port_5196_shared_use_temp_config
description: "ai-chat-admin-frontend Playwright webServer is fixed to 5196 with reuseExistingServer:false; a parallel session's run holds it → \"already used\". Use an uncommitted config on another port, never reuse/kill the other server"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9bfbf442-2ea7-482e-8e36-8246c00e89fd
  modified: 2026-11-02T21:02:46.363Z
---

Seen 2026-11-03 (AI-371, while AI-373 ran e2e): `playwright test` → "http://localhost:5196 is already used". Reusing would test another branch's code.

**How to apply:** in `apps/admin`, an untracked `playwright.<card>.config.ts`:
`import base from "./playwright.config"; export default {...base, use:{...base.use, baseURL:"http://localhost:5197"}, webServer:{...base.webServer, url:"http://localhost:5197", env:{...base.webServer.env, PORT:"5197"}}}` (vite reads `PORT`). Run `./node_modules/.bin/playwright` from `apps/admin` (the root `node_modules/.bin` has none), unsandboxed. Delete the config after. Unsandboxed `$TMPDIR` differs ([[reference_tmpdir_differs_sandboxed_vs_unsandboxed_bash]]) — write logs to the scratchpad path.

**Simpler (2026-11-05, wave 3):** no temp config needed — `apps/admin/e2e/mockServer.mjs` reads `E2E_PORT` (default 5196) for both the vite server and baseURL. `E2E_PORT=5198 npx playwright test …` from `apps/admin`, unsandboxed. Check the port is free with `nc -z localhost <port>` first.
