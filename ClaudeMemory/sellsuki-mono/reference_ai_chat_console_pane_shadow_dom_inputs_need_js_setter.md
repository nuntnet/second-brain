---
name: reference_ai_chat_console_pane_shadow_dom_inputs_need_js_setter
description: Driving the AI Chat console in the Claude browser pane (2026-10-28) — DS inputs live in shadow DOM (ssk-textarea/ssk-button), so coordinate clicks and `type` never reach them; set the value via the native setter + composed `input` event and click the inner <button>; the pane's network/console capture records nothing for fetch, so read the DOM and call chat-core from the page with credentials include
metadata:
  type: reference
---

Verified 2026-10-28 on https://ai-chat-admin.staging-th.sellsuki.com (Sprint 21 verifier pass, AI-335/336/345):

- `find`/`read_page interactive` do not list the KB playground textarea or submit button: they are inside
  `ssk-textarea` / `ssk-button` shadow roots. Pane `left_click` + `type` left the field empty 3 times.
- What works: walk shadow roots, then
  `Object.getOwnPropertyDescriptor(HTMLTextAreaElement.prototype,'value').set.call(ta, text)` +
  `ta.dispatchEvent(new Event('input',{bubbles:true,composed:true}))` → React state updates (button enables);
  submit with `sskButton.shadowRoot.querySelector('button').click()`.
- Screenshot frame is 800×600 for an emulated 1280×900 viewport: multiply page coords by 0.625.
- `read_network_requests` listed only the document GET and `read_console_messages` returned nothing for the
  whole drive, so an AC that says "logs the cause to console" cannot be judged in the pane. Evidence came from
  DOM reads (`role=alert`) and from `fetch(chat-core.staging-th…/v1/…, {credentials:'include'})` run inside the page.
- An injected failure is cheap: wrap `window.fetch` to answer 503 for one path, then reload to undo.

How to apply: for any console verification in the pane, start with the JS-walk helper instead of coordinates, and
plan DOM/API readbacks as the evidence rather than network/console captures. Related:
[[reference_staging_ai_chat_console_drive_recipe]], [[feedback_verify_as_the_user_sees_it]].
