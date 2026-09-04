---
name: reference_i18next_mutates_the_json_you_import
description: i18next keeps a reference to the resources passed to init() and addResource writes through it, so patching a key corrupts the imported locale JSON
metadata:
  type: reference
---

`i18n.init({ resources: { th: { translation: th } } })` stores the **object it
was handed**, not a copy. `addResource(lng, ns, key, value)` then does
`setPath(this.data, …)` — no copy anywhere — so any runtime patch mutates the
imported `th.json` module itself. Every other importer of that module now sees
the patched value.

Bit us in AI-207 (label pack): the overlay patched `admin.shell.nav_leads`, and
`shippedLabel()` — which reads the JSON directly to show an admin the ORIGINAL
word beside their override — started answering the override. The settings screen
would have shown the admin their own rename in the "original" column, and
withdrawing a rename would have restored the rename.

Fix: `structuredClone` the bundle behind any code that needs the shipped
strings (`apps/admin/src/i18n/shipped.ts`).

Two more i18next facts from the same work:
- `addResources(lng, ns, flatMap)` with **dotted keys** is the right primitive
  for patching individual leaves — it calls `addResource` per entry and
  preserves siblings. `addResourceBundle` with a nested object and `deep=false`
  shallow-merges and **blanks the siblings**.
- react-i18next's default `bindI18nStore` is `''`, so components never
  re-render on `addResource`. Set `react: { bindI18nStore: "added" }`, and note
  effects in one commit run before sibling components subscribe — an inline
  `changeLanguage` re-emit reaches nobody, so queue it as a microtask.

Related: [[reference_ds_1_0_beta_gotchas]] · [[reference_ds_testid_is_a_property]]
