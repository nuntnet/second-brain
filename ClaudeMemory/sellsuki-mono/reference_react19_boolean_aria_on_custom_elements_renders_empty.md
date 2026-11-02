---
name: reference_react19_boolean_aria_on_custom_elements_renders_empty
description: "React 19 writes a boolean `true` prop on a custom element (ssk-*) as an EMPTY attribute and drops `false`: aria-pressed={x} becomes aria-pressed=\"\" (invalid ARIA). ai-chat-admin uses `ariaBool()` (lib/ariaBool.ts) on every boolean ARIA prop of an ssk-* host"
metadata:
  type: reference
---

Found 2026-11-02 moving ai-chat-admin-frontend to React 19 (AI-118, branch `feature/AI-118-frontend-kit`):
17 unit tests went red with `expected '' to be 'true'` on `getAttribute("aria-pressed" | "aria-selected")` of
`ssk-button` / `ssk-tab-button` hosts. React 18 stringified booleans on custom elements; React 19 treats them like
boolean HTML attributes (present-empty / absent). TypeScript does not catch it.

**How to apply:** on any `<Ssk*>` / `ssk-*` element pass boolean ARIA as `"true" | "false"` strings
(`ariaBool(v)` in that repo). New code written on React 18 (e.g. AI-370's `ViewTabs` `aria-selected={active}`)
must be converted when it lands on the React 19 branch. Plain HTML elements are unaffected.
