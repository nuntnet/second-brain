---
name: reference_ssk_drawer_named_slots_and_closed_header_h1
description: "ssk-drawer paints ONLY named slots (body content needs slot=\"body\" or it is in the DOM but invisible; jsdom passes) and a CLOSED drawer still exposes its DS header as an h1 (two h1 per page) — ai-chat-admin SideDrawer now renders null when closed"
metadata:
  node_type: memory
  type: reference
  originSessionId: b5b359bf-6b73-4b00-807e-134a240abc51
  modified: 2026-11-02T20:53:16.585Z
---

Found 2026-11-03 (AI-372, console `feature/AI-372-g3-customers-board` 5437925 / cec03d4):

- `<ssk-drawer>` has slots `header` / `body` / `footer` only. AI-370's `SideDrawer` put children in a plain div, so every drawer
  body (customers drawer, AI-370 end-systems drawer) rendered blank in a real browser while all jsdom tests passed — only the
  e2e / a screenshot showed it. Same for `ssk-modal`.
- A closed drawer's `SskDrawerHeader` is still in the a11y tree as `heading level=1`, so `shell-semantics.spec.ts` "exactly one
  h1" fails on any page that mounts a closed drawer. Fix: render nothing when `!show`.

**How to apply:** for any `ssk-drawer`/`ssk-modal` content, check `slot=`; for a page with a drawer, open it in a browser (or
Playwright screenshot) before calling it done. Related: [[reference_ds_ssk_drawer_closes_on_escape_itself]].
