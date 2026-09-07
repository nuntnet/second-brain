---
name: reference_daisyui_progress_class_collision
description: OC2Plus FE uses DaisyUI whose global component classes (.progress etc.) silently collide with your own scoped class names
metadata: 
  node_type: memory
  type: reference
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-09-07T04:51:09.256Z
---

**DaisyUI ships GLOBAL component classes that collide with your own `class="…"`
names in the OC2Plus frontends** (member + backoffice both use DaisyUI). Verified
2026-09-07 fixing a real bug.

`frontend/oc2plus-linecrm-frontend-member` `PointClaimNewView.vue` used
`<ol class="progress">` for a step list. DaisyUI defines `.progress` as a
progress-**BAR**: `height: 0.5rem (8px)`, `overflow: hidden`, a background, rounded.
The component's OWN scoped `.progress[data-v-…]` rule set `display/flex/gap` but
**not** height/background/overflow — so for those properties DaisyUI's global
`.progress` won unopposed, collapsing the `<ol>` into an **8px clipped blue bar**
with its text squashed/overlapping. Looked like a broken success screen; the code
was "correct" and grep of the file showed nothing wrong. **Fix: rename the class**
(`.progress` → `.claim-steps`, template + scoped CSS). BEM children (`.progress__step`)
don't collide — only the bare DaisyUI names do.

**How to diagnose this class of bug:** the source looks fine, so inspect the LIVE
element's *computed* style (`getComputedStyle`) — a background/height/overflow you
never wrote = a global (DaisyUI/Tailwind) class of the same name winning. To reach a
hard-to-trigger view state for inspection, set the Vue ref directly through the
component proxy: `el.__vueParentComponent.setupState.<ref> = value` (setupState
auto-unwraps refs, so assign the value, NOT `.value`).

**Rule:** never name a scoped class after a bare DaisyUI component
(`progress`, `badge`, `card`, `alert`, `steps`, `tab`, `toggle`, `range`, `menu`,
`modal`, `drawer`, `join`, `stat`, `loading`, `divider`, …). Prefix your own
(`claim-steps`, `pc-badge`). Same family as [[reference_ds_1_0_beta_gotchas]] but
for DaisyUI, not the ssk-* DS. Committed fix: member FE `1cec544`.
