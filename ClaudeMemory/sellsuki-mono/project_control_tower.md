---
name: project-control-tower
description: Sellsuki Control Tower website — self-contained visual ecosystem/roadmap dashboard at docs/control-tower/
metadata: 
  node_type: memory
  type: project
  originSessionId: 06c2449b-af9f-42b9-bf19-e32fb331ac0f
---

The user built a **Control Tower website** visualizing the entire Sellsuki software ecosystem, roadmap, progress, and product connections. Lives at `docs/control-tower/` in `sellsuki_mono` (monorepo root, not a submodule).

- **Files:** `index.html` (single-file, no build step — embedded CSS+JS) + `data.js` (all data baked in one object `window.CT`).
- **Run it:** preview config `control-tower` in `.claude/launch.json` (`python3 -m http.server 4480` in that dir), or open index.html directly.
- **Sections:** Overview (stats) · Ecosystem (interactive SVG hub-and-spoke connection map + product cards) · Services (42 filterable cards) · Roadmap (9 themes, collapsible, with progress bars) · Timeline (3 integration phases).
- **Data sources (baked, refresh by editing data.js):** `docs/sellsuki-roadmap.md`, `docs/sellsuki-ecosystem-overview.md`, `.claude/CLAUDE.md`, and **roadmap progress/Jira-card status is derived from `.claude/knowledge/epic-index.md`** (the repo's PO source-of-truth, updated periodically). Progress scoring rule: done=100% · in-progress=50% · to-do(carded)=15% · no-card=0%.
- **Design:** dark control-tower aesthetic (bg #020617, Fira Sans + Fira Code) — same tokens as the [[project-helio]] dashboard.
- Jira base URL for card links = `https://sellsuki.atlassian.net`.
- Not yet committed as of 2026-06-25 (was on unrelated branch `feat/qms-api-test-suite-oc4275`). User may want a live-data version (pull card status from Jira/MCP) instead of baked-static later.

**Env gotcha:** the Claude Preview screenshot tool in this workspace gets stuck capturing at scroll-0 after several reloads/resizes — verify deep-page sections via DOM/computed-style (preview_eval) instead of screenshots.
