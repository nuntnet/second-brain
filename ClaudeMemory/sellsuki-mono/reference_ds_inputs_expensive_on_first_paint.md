---
name: reference_ds_inputs_expensive_on_first_paint
description: A few hundred DS (Lit) inputs on one screen costs seconds per render — a list screen must not render its whole catalogue by default
metadata:
  type: reference
---

`ssk-*` components are Lit custom elements, and they are **not cheap in bulk**.
The AI-207 Words screen rendered 116 rows × 2 `SskInput` = **232 components** on
first paint and re-rendered all of them on every keystroke in its filter box.

Measured in `apps/admin`: that one test file took **32 s**; capping the default
row set brought it to **0.7 s** — a 45× difference from render volume alone.

Consequences worth remembering:
- A settings/list screen must **not** render its whole catalogue by default.
  Show what the user has actually touched, and reach the rest by search.
- The symptom in CI is a *slow* test file, and under parallel load
  (`pnpm -r test`) it becomes **failing** tests that pass when run alone — easy
  to misread as flake. Check the duration before blaming timing.
- It is a product defect before it is a test problem: a real admin experiences
  it as a page that freezes while they type.

Related: [[reference_ds_bundle_dominates_and_cannot_treeshake]] ·
[[reference_chatcore_route_workspace_flaky_under_load]]
