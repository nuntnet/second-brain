---
name: ai-chat-frontend-design
description: "The AI chat frontend's real design system lives in a Claude Design project — and it conflicts with what ai-chat-admin-frontend was built against"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-11T11:33:21.208Z
---

The design source of truth for the AI chat frontend is a **Claude Design project**, not a Figma link or a repo:
`https://claude.ai/design/p/779693b7-606b-40b9-85cf-6e5c78d84e80` — *"ออกแบบ ui สำหรับ Saas chat interface for business which can support desktop, mob"*. Read it with the `DesignSync` tool (`get_project` / `list_files` / `get_file`), not WebFetch.

Contents: `Chat Platform UI v2 SSK.dc.html` + `Chat Platform UI.dc.html` (the screens), `colors_and_type.css` (full token set), self-hosted `fonts/DBHeaventRounded-*.woff2` ×4, and a copy of the platform plan under `uploads/`.

**Three hard rules the CSS states explicitly:**
1. **DB HeaventRounded is the only typeface** across every Sellsuki product surface — "Never substitute Inter / Roboto / system-ui".
2. **No font size below 18px anywhere on screen**; body/label = 20px, data minimum 20px, buttons/captions 18px. This is far larger than typical and reshapes table/sidebar/card layouts.
3. Brand primary is Sky-500 `#32a9ff`; per-product themes switch only `--primary` and `--sidebar-accent-*` via `data-product="patona|akita|…"`; `--background` never re-themes. Layout constants: navbar 56px, sidebar 200px (64px collapsed), max content 1280px.

**RESOLVED 2026-08-11 — the design artifact is the one that's wrong, not the app.** `@sellsuki-org/sellsuki-components` (Lit, v0.27) is authoritative for all new frontends: plan **D7** names it explicitly, and epic **PAT-2587** (the frontend-kit spec, Patona-owned) lists "Lit DS" / "RN mapping ของ sellsuki-components". **`@uxuissk/design-system` is legacy** — a companion token package still pinned by Svelte-era apps (`sellercenter-frontend`, `system-management`, `shipmunk-frontend/packages/ui`), and it appears nowhere in PAT-2587 or the platform plan. `colors_and_type.css`'s claim that its bridge tokens "match the shipping @uxuissk/design-system" points at the wrong package and should be corrected in the design.

**Still open — typography conflict.** PAT-2587 and the plan say **nothing** about fonts or a minimum size; D7 says the DS owns type/spacing tokens and forbids hardcoded font-size. So the design's "DB HeaventRounded only / no text under 18px / body 20px" has no backing in the engineering spec, and hardcoding it locally would violate D7. Needs a human decision: does the design doc override, or does the DS?
