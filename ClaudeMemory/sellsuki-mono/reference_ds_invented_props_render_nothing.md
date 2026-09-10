---
name: reference_ds_invented_props_render_nothing
description: An ssk-* prop that does not exist in sellsuki-components fails silently — errormessage swallowed every inline form error on two pages; check the prop against the DS and against working call sites before trusting it
metadata:
  type: reference
---

`ssk-input` has **no `errormessage` attribute.** The DS exposes `helperText`, `error` and `status`; the string "errormessage" appears **nowhere** in `sellsuki-components`.

Writing `errormessage={msg}` on a custom element sets a meaningless attribute. Nothing throws, nothing type-errors, `svelte-check` stays at baseline — and **nothing ever renders.** On CCS3's two BOLA pages that silently swallowed every inline error in four flows (create workspace, rename, invite, add member): validation messages and server refusals alike. Found 2026-09-10 only because a person kept saying "ไม่ขึ้น alert เลย" three times running.

**How to tell before shipping:** count the call sites. `errormessage` had 3, all in the two new pages. The working convention has **25** — `helperText={msg}` paired with `error={!!msg}`, as in `components/role/NameRoleForm.svelte` and `components/form/AddressForm.svelte`. A prop nobody else in the app uses is a prop nobody has verified. This also applies to props that DO exist in the DS types: `status: "error"` is real but unused here, so prefer the shape the app actually runs.

Casing is not the trap — the repo uses `themecolor` and `themeColor` about equally (126/116) because the DOM lowercases attributes on custom elements either way.

**Second, independent trap on the same bug:** this app's modals scroll their body (`src/utils/modal-scroll.ts`, selector `[data-modal-scroll]`). An error region appended at the END of a tall dialog body renders correctly and still reads as "nothing happened", because it is below the fold. Put it at the top of the body, and for a failed request also raise a toast — that is the app's own convention (`UserGroup`, `Pay`, `User/TabListInvite`) and it cannot land off-screen. Verify the route sits inside `<ssk-toast-provider>` in `App.svelte` before relying on a toast.

Related: [[reference_ds_testid_is_a_property]] · [[reference_ds_1_0_beta_gotchas]] · [[feedback_check_the_norm_before_calling_it_broken]]
