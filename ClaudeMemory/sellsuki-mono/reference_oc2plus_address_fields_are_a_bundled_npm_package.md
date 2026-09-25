---
name: reference_oc2plus_address_fields_are_a_bundled_npm_package
description: "OC2Plus address dropdowns call no API — thai-address-database ships the whole 7,480-row table in the frontend bundle; maxResult defaults to 20 and truncates silently, and the data is substitution-encoded so grepping province names proves nothing"
metadata:
  type: reference
---

`CreateCompany.vue` / `CreateDpa.vue` in `frontend/oc2plus-linecrm-frontend-backoffice`
resolve จังหวัด/อำเภอ/ตำบล/รหัสไปรษณีย์ from the npm package
**`thai-address-database`**, bundled client-side. **There is no address API and
nothing to seed** — "ไม่มี seeding data หรอ" is the wrong question and cost an hour
on 2026-09-24.

Shape: 7,480 full-address rows, **5,879 distinct tambon names**, 77 provinces. Every
row carries province + amphoe + district + zipcode, so one tambon pick fills a whole
address (this is why OC-4621 made ตำบล the only searched field).

**Three traps, all of which mislead silently:**

- **`maxResult` defaults to 20 and drops the rest with no signal.** `ในเมือง` really
  matches 22 places; you see 20. `หนองบัว` exists in 28. Always pass a cap, and ask for
  cap+1 so you can TELL the user the list was cut.
- **An empty `searchStr` returns `[]` by design**, and `searchStr` is used as a
  **RegExp** — so `(` or `[` from a real keystroke throws. Wrap the call.
- **`database/db.json` is substitution-encoded** (`Iท่อม`, `MQขาว`, numeric
  back-references). `เชียงใหม่` does not appear literally even in the source file, so
  grepping a deployed bundle for a province name and finding nothing proves NOTHING.
  Probe with the real signature instead: `กระบี่` / `Iท่อม` / `81120`.

Vocabulary does not match the form labels, and reading it wrong is the bug this note
exists to prevent: **`district` = ตำบล/แขวง (the smallest unit)**, `amphoe` = อำเภอ/เขต.
A TypeScript declaration with these fields lives at
`src/types/thai-address-database.d.ts` (added OC-4621) — the package ships no types, so
a `.ts` file importing it errors TS7016 while a `.vue` file does not.

Related: [[reference_ds_invented_props_render_nothing]] — the same screen's dropdowns
bound a `:hideOptions` prop that does not exist.
