---
name: ai-chat-frontend-design
description: "The AI chat frontend's design sources — the Claude Design project, and where it now disagrees with the shipping design system"
metadata:
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-11T12:04:57.575Z
---

The design source of truth for the AI chat frontend is a **Claude Design project**, not a Figma link or a repo:
`https://claude.ai/design/p/779693b7-606b-40b9-85cf-6e5c78d84e80`. Read it with the `DesignSync` tool (`get_project` / `list_files` / `get_file`), not WebFetch.

Contents: `Chat Platform UI v2 SSK.dc.html` + `Chat Platform UI.dc.html` (the screens), `colors_and_type.css` (full token set), self-hosted `fonts/DBHeaventRounded-*.woff2` ×4, and a copy of the platform plan under `uploads/`.

**The design artifact is out of date on two counts — the app is not wrong.**

1. **Typeface — settled, don't re-litigate.** The design CSS says DB HeaventRounded is the only typeface ("Never substitute Inter / Roboto / system-ui"). The DS moved the other way: **DS 1.0 (`0.27.0-beta.1`) replaced it with Noto Sans Thai**, embedded base64 in the bundle. Source-verified in the tag — `src/assets/fonts.css` declares it and `theme/default.ts` says "type to bind DS1.0 to Noto Sans Thai". Since D7/PAT-2587 give the DS ownership of type tokens, the DS wins; `ai-chat-admin-frontend` consumes `var(--ssk-font-family-sans)` and pins no font locally. **The design doc should be corrected at the design end.**
2. **Wrong package named.** `colors_and_type.css` claims its bridge tokens match `@uxuissk/design-system` — that's the legacy Svelte-era package. `@sellsuki-org/sellsuki-components` is authoritative.

**Still genuinely open (needs a human):** the design's **18px floor** (body/label 20px, buttons/captions 18px). DS 1.0's own scale defines `body` 16 / `body-sm`+`label` 14 / `caption` 12, so honouring the floor means overriding DS tokens, which D7 forbids. Deliberately not worked around in the app.

Other layout constants from the design, unaffected: Sky-500 `#32a9ff` primary; per-product themes switch only `--primary` and `--sidebar-accent-*` via `data-product`; navbar 56px, sidebar 200px (64px collapsed), max content 1280px.

See [[ds-1-0-beta-gotchas]] for the version/token traps hit while adopting it.

**IMPLEMENTED 2026-08-14** (commit 966885d on `feature/AI-122-flags-port`): the
handoff's screen 2a + 3a segmented control now IS the inbox
(`apps/admin/src/pages/inbox/`, styles in `inbox.css`), brand switched
patona→ccs3 (sky). The 18px floor stays unadopted. DS gotchas hit while
implementing: `<ssk-heading>` renders section-title scale even at
`size="sm"` (use a token-sized span for navbar/sidebar labels), and
`<ssk-divider>` renders as a thick gray block inside `<ssk-sidebar>` (use a
1px `--stroke-primary` hr). Live refs from chat-core are `facebook:<psid>`,
not the `fb:psid:` shape AI-101's mocks used.
