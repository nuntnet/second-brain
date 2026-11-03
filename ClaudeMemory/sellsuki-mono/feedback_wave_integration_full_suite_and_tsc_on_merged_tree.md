---
name: feedback_wave_integration_full_suite_and_tsc_on_merged_tree
description: "Wave integration of the console - run tsc and the FULL e2e suite on the merged tree; card \"e2e green\" covered touched specs only, vitest does not type-check"
metadata:
  node_type: memory
  type: feedback
  originSessionId: f746364c-7c1d-4189-93bc-f067e13be78d
  modified: 2026-11-03T17:02:07.048Z
---

Integrating redesign wave 1 into one console branch (2026-11-03/04, `feature/redesign-v9-wave1` @ `edf1cbb`):

- **3703 unit tests green while `tsc` and the build failed.** vitest does not type-check. Two cards exported different `LineGroup` types from `@admin/core` (TS2308), visible only in the merged tree. Always run `pnpm type-check` + `pnpm lint` + build, not just `pnpm test`.
- **Every card reported "e2e green", yet the full suite had 15 failures, 14 of them already on the card heads alone.** Their runs covered the specs they touched. One was a real defect (selected chip contrast) that the spec would have caught had it been pointed at the new UI.
- **How to attribute a failure:** detach-worktree each card head (own `pnpm install --offline`, own port config) and run only the failing spec files there. Fails there = card's own; passes there = the merge.
- Playwright CSS pierces shadow roots by itself; `>>>` is axe's syntax and times out in Playwright. Prefer `getByRole(name)`.
- a11y baseline policy (console `e2e/a11y-baseline.json`): DS-owned shadow-root targets go in with a reason; app-owned ones are fixed in the app. A rule without a reason fails validation; add the reason only if it really is DS-owned (button-name: ssk-dropdown-button does not forward aria-label).

**Why:** "unit green" and "the cards were green" both read as done and were not.
**How to apply:** for any multi-card integration branch, the gate is tsc + lint + build + unit + FULL e2e on the merged head, and the morning report says which rungs ran. Related: [[reference_merge_conflict_keep_both_drops_shared_closing_lines]], [[reference_console_e2e_port_5196_shared_use_temp_config]], [[feedback_open_mr_only_after_full_test_ladder_green]].
