---
name: reference_first_ci_run_of_never_run_playwright_specs_finds_real_bugs_and_axe_baseline_drift
description: "ai-chat-admin-frontend 2026-10-24: Playwright specs written in a sandbox that cannot bind ports fail on their first CI run for the same few reasons — DS testId is a property not an attribute, ssk-button data-testid matches host AND inner button (strict mode), route swap detaches the header, Thai font differs on CI, axe baseline from a desktop Chromium differs from CI. Never rebase a pushed branch to fix conflicts — merge main. Main e2e needs the staging-th runner tag so it cannot run 21:00-09:00."
metadata:
  node_type: memory
  type: reference
  originSessionId: 6f33fae2-e867-41c2-ba6c-f51697e817e5
  modified: 2026-10-24T14:10:06.572Z
---

Seen on the UXC set (AI-304..311), MRs !107-!114 of `frontend/ai-chat-admin-frontend`.

- **Specs never ran locally** (sandbox cannot bind a port), so the first CI run is the first run. Budget a fix commit per MR.
- `[data-testid="x"]` on a DS component matches nothing for `ssk-range-date-picker` (testId is a property, not reflected); select the element (`ssk-range-date-picker`) instead.
- `getByTestId("composer-send")` on an `ssk-button` matched two nodes (host + inner `<button>`): strict-mode violation. Use `ssk-button[data-testid="…"]`.
- A route swap can detach the header between `toBeVisible` and `evaluate` (empty computed style, zero controls): `expect.poll` on `isConnected`.
- Thai geometry depends on the installed font; CI's Thai face is not a designer's Mac. Geometry assertions about Thai chip widths are not portable.
- **axe baseline** captured on a desktop Chromium was wrong on CI (52 scenes, +262/-56 targets). Fix = apply exactly the added/fixed lines the failing assertions print, as a new commit. Merging other cards later drifts it again; each MR re-applies its own diff and the JSON conflicts.
- Removing a tinted box moved text onto `--bg-secondary`, where `--text-secondary` is 4.39:1 (axe flagged 39 scenes). Check contrast when a background goes away.
- **Never rebase a branch that is already pushed** (no force push rule): merge `origin/main` into it. Rebase is fine only before the first push.
- Main pipelines for this repo need `staging-th` runners: nothing runs 21:00-09:00. Overlapping main pipelines produce `failed_outdated_deployment_job` on the newest one; retry the failed jobs by id in the morning and verify by the pod image.
- `deploy_staging` jobs are `manual`; merging to main does not deploy. chat-core: play `deploy_staging_th_arm`, then check `kubectl -n sellsuki get deploy sellsuki-chat-core -o jsonpath` for the image sha.
- Confirm an API field on staging without a token: `fetch` from the logged-in browser pane (`credentials: "include"`) to `https://chat-core.staging-th.sellsuki.com/v1/workspaces/<ws>/conversations?limit=50`; report counts, not message text.
