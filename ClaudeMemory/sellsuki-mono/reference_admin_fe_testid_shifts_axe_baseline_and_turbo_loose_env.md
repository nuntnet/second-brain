---
name: reference_admin_fe_testid_shifts_axe_baseline_and_turbo_loose_env
description: ai-chat-admin console — adding testId to an ssk-* element changes its axe selector and reds e2e a11y-baseline; sandboxed turbo test needs --env-mode=loose; read CI job trace via unsandboxed glab
metadata:
  node_type: memory
  type: reference
  originSessionId: 42a0336a-1640-4f81-9472-135132c8e7f9
  modified: 2026-10-29T10:36:42.281Z
---

Three traps hit on 2026-10-29 (AI-316, MR !130) in `frontend/ai-chat-admin-frontend`:

1. **testId on a DS element moves its a11y baseline entry.** Adding `testId` to an `<ssk-tag>` makes axe report a pre-existing violation under a new target (`… >>> span[data-testid="…"]` instead of `… >>> span`). `e2e/a11y.spec.ts` then fails with one "added" plus one "fixed" entry. Unit tests stay green, so this only shows in the CI e2e job. Fix: update `apps/admin/e2e/a11y-baseline.json` to the exact target the job prints, for th and en. When you add a testId to an ssk-* element, grep the baseline for its old selector before pushing.
2. **Sandboxed `turbo run test` fails with `EPERM mkdir /tmp/…`.** Turbo's strict env mode drops TMPDIR. Run it as `turbo run type-check lint test --force --env-mode=loose`. Creating the worktree and running `pnpm install` both need to run unsandboxed: `.claude/commands` is write-protected, and the sandbox blocks some package dirs such as `.idea`.
3. **Read a CI job trace with unsandboxed `glab api projects/<url-encoded path>/jobs/<id>/trace`.** In the sandbox it fails TLS (x509 OSStatus), and rtk mangles glab output. GitLab accepts the URL-encoded project path in place of a numeric id, which also works in create-mrs scripts.

Local Playwright for this repo wants chromium build 1234; only 1248 is installed (2026-10-29), so local e2e does not run until `npx playwright install` is done. See [[reference_admin_fe_worktree_borrowed_node_modules_test_main_core]], [[reference_auto_mode_blocks_glab_mr_create]].
