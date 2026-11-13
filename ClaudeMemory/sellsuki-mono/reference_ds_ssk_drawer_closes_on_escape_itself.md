---
name: reference_ds_ssk_drawer_closes_on_escape_itself
description: "DS ssk-drawer (0.27.0-beta.1) fires its own `close` on Escape — an app Escape listener closes it twice, and a modal over it closes both (fix: modal takes Escape in window capture); raw Event('input') on ssk-textarea hangs jsdom; ssk-tooltip has no public open/close"
metadata:
  type: reference
---

Observed 2026-11-02 in ai-chat-admin-frontend unit tests that run the real Lit DS (AI-370 `SideDrawer`): with a
document Escape listener added by the app, one Escape produced two `onClose` calls. The drawer's own
`close` (the `onClose` mapped in `lib/ds.tsx`) already covers Escape, the backdrop and the header's X. So an app
drawer wrapper should add only what the DS lacks (swipe-to-close), not Escape.

`ssk-tooltip` in the same DS version: `trigger` is `"hover" | "click"`, and there is no public open/close. Closing
on an outside tap / Escape / opening on keyboard focus goes through the private `_isOpen` + `requestUpdate()`
(as the AI-369 kit mock does). Treat it as a DS gap and keep it in one helper.

**A modal opened over a drawer** (2026-11-14, AI-49 reopen dialog over the customer drawer): one Escape closed BOTH
the ssk-modal and the ssk-drawer. Fix that worked: the dialog adds a `window` keydown listener in the capture phase
while open, and on Escape calls its own cancel + `preventDefault()` + `stopImmediatePropagation()`
(`StageChangeDialog.tsx`). jsdom reproduces it (`fireEvent.keyDown(document, {key:"Escape"})` → drawer gone).

Also: a raw `el.dispatchEvent(new Event("input", {bubbles:true}))` on an `ssk-textarea` host never returns in jsdom
(the test hangs to its 5 s timeout, no assertion error); `fireEvent.input(el)` works.

Related: [[reference-ds-testid-is-a-property]] · [[reference_ssk_drawer_named_slots_and_closed_header_h1]]
