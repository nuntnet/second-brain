---
name: ai-chat-frontend-design
description: "The AI chat frontend's real design system lives in a Claude Design project — and it conflicts with what ai-chat-admin-frontend was built against"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-11T10:37:42.724Z
---

The design source of truth for the AI chat frontend is a **Claude Design project**, not a Figma link or a repo:
`https://claude.ai/design/p/779693b7-606b-40b9-85cf-6e5c78d84e80` — *"ออกแบบ ui สำหรับ Saas chat interface for business which can support desktop, mob"*. Read it with the `DesignSync` tool (`get_project` / `list_files` / `get_file`), not WebFetch.

Contents: `Chat Platform UI v2 SSK.dc.html` + `Chat Platform UI.dc.html` (the screens), `colors_and_type.css` (full token set), self-hosted `fonts/DBHeaventRounded-*.woff2` ×4, and a copy of the platform plan under `uploads/`.

**Three hard rules the CSS states explicitly:**
1. **DB HeaventRounded is the only typeface** across every Sellsuki product surface — "Never substitute Inter / Roboto / system-ui".
2. **No font size below 18px anywhere on screen**; body/label = 20px, data minimum 20px, buttons/captions 18px. This is far larger than typical and reshapes table/sidebar/card layouts.
3. Brand primary is Sky-500 `#32a9ff`; per-product themes switch only `--primary` and `--sidebar-accent-*` via `data-product="patona|akita|…"`; `--background` never re-themes. Layout constants: navbar 56px, sidebar 200px (64px collapsed), max content 1280px.

**Unresolved conflict as of 2026-08-11:** `colors_and_type.css` says its bridge tokens "match the shipping **`@uxuissk/design-system`**", but `frontend/ai-chat-admin-frontend` was built against **`@sellsuki-org/sellsuki-components` v0.27** (Lit) with `--ssk-*` tokens and `system-ui` as the font fallback. Token names also differ (`--bg-primary`/`--fg-primary`/`--stroke-primary` + bridge `--background`/`--foreground`/`--primary` vs `--ssk-*`). Before doing more admin-frontend UI work, confirm which package is real and whether the design covers only the chat/inbox screen or the whole shell — nothing in that repo is merged yet (`main` is still just README + .gitignore), so this is the cheapest possible moment to switch. See [[project-ai-sprint234-autonomous-run]] for the branch map.
