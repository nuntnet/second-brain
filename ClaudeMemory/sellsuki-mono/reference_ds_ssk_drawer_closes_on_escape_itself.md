---
name: reference_ds_ssk_drawer_closes_on_escape_itself
description: "DS ssk-drawer (0.27.0-beta.1) already fires its own `close` event on Escape — an app-level Escape listener on top closes it twice; ssk-tooltip has no public open/close (only the private `_isOpen`)"
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

Related: [[reference-ds-testid-is-a-property]]
