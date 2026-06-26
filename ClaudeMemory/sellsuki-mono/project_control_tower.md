---
name: project-control-tower
description: Sellsuki Control Tower website — self-contained visual ecosystem/roadmap dashboard at docs/control-tower/
metadata: 
  node_type: memory
  type: project
  originSessionId: 06c2449b-af9f-42b9-bf19-e32fb331ac0f
---

The user built a **Control Tower website** visualizing the entire Sellsuki software ecosystem, roadmap, progress, and product connections. Lives at `docs/control-tower/` in `sellsuki_mono` (monorepo root, not a submodule).

- **Files:** `index.html` (single-file, no build step — embedded CSS+JS) + `data.js` (all data baked in one object `window.CT`) + `server.py` (static server + RAG proxy).
- **Run it:** preview config `control-tower` in `.claude/launch.json` → `python3 server.py 4480` in that dir. (Plain `python3 -m http.server` works for everything EXCEPT the Ask/RAG panel, which needs the proxy.)
- **Live Jira roadmap status:** roadmap *structure* stays baked in data.js; *card status + progress* refresh live. Browser → same-origin `POST /api/jira-status {ids:[...]}` → server.py Basic-auth (jira_email+jira_token from config.local.json) → `POST sellsuki.atlassian.net/rest/api/3/search/jql {jql:"key in(...)",fields:["status"]}` → maps statusCategory `done→done · indeterminate→wip · new→todo` → frontend overlays onto baked cards, recomputes progress, re-renders, shows "● live · Jira · as of <time>" badge (else "snapshot"). server.py = ThreadingHTTPServer (single-thread froze whole site on slow RAG) + `Cache-Control:no-store` + 60s RAG / 30s Jira timeouts. GOTCHA: `.ask-panel{display:flex}` beat UA `[hidden]{display:none}` (equal specificity, author wins) → panel wouldn't hide; fixed with `.ask-panel[hidden]{display:none}`.
- **Ask the platform (Sellsuki RAG):** floating "✦ Ask" panel queries Sellsuki RAG. Browser → same-origin `POST /rag/ask` → `server.py` injects `X-API-Key` server-side → `https://api-rag.dev-th.sellsuki.com/v1/api/chat/ask` (channel="web", needs session_id/message_id/idempotency_key/query). RAG answers in Thai + citations. **Why proxy:** RAG gateway returns 405/no-CORS on preflight → browser can't call it cross-origin with X-API-Key; AND the key must not ship to the client. Key lives in `config.local.json` (gitignored; server.py also 404s it + dotfiles over HTTP). `config.example.json` is the committed template. SSL needs certifi (`ssl.create_default_context(cafile=certifi.where())`) — macOS system Python lacks CA bundle. MCP equivalent: `https://api-rag.dev-th.sellsuki.com/mcp` ("Channel Gateway MCP", needs X-API-Key + session).
- **Sections:** Overview (stats) · Ecosystem (interactive SVG hub-and-spoke connection map + product cards) · Services (42 filterable cards) · Roadmap (9 themes, collapsible, with progress bars) · Timeline (3 integration phases).
- **Data sources (baked, refresh by editing data.js):** `docs/sellsuki-roadmap.md`, `docs/sellsuki-ecosystem-overview.md`, `.claude/CLAUDE.md`, and **roadmap progress/Jira-card status is derived from `.claude/knowledge/epic-index.md`** (the repo's PO source-of-truth, updated periodically). Progress scoring rule: done=100% · in-progress=50% · to-do(carded)=15% · no-card=0%.
- **Design:** dark control-tower aesthetic (bg #020617, Fira Sans + Fira Code) — same tokens as the [[project-helio]] dashboard.
- Jira base URL for card links = `https://sellsuki.atlassian.net`.
- Not yet committed as of 2026-06-25 (was on unrelated branch `feat/qms-api-test-suite-oc4275`). User may want a live-data version (pull card status from Jira/MCP) instead of baked-static later.

**Env gotcha:** the Claude Preview screenshot tool in this workspace gets stuck capturing at scroll-0 after several reloads/resizes — verify deep-page sections via DOM/computed-style (preview_eval) instead of screenshots.
