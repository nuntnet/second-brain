---
name: reference_admin_fe_worktree_install_and_i18n_key_count_tripwire
description: ai-chat-admin-frontend worktree needs its own pnpm install; main checkout can be 64 commits behind origin/main; LabelPackSettingsPage test counts i18n keys
metadata:
  type: reference
---

- A new worktree of frontend/ai-chat-admin-frontend has no node_modules: `pnpm install --offline --frozen-lockfile` (~25s, unsandboxed) gives it its own @admin/* packages. The main console checkout may sit far behind origin/main (64 commits on 2026-10-30), so it is not a baseline: make a throwaway worktree of origin/main to check whether a failing test is pre-existing.
- `apps/admin/src/pages/settings/LabelPackSettingsPage.test.tsx` asserts `"count":N` = number of i18n keys under overridable prefixes (admin.lead.* etc). Adding or removing keys there reds it; update N with a comment naming the keys (140 -> 151 on AI-355).
- Under a full `turbo run test` ~10 unrelated KB/inbox tests flake on load and pass alone; rerun those files individually before blaming your diff.
- `orval` from the root: `./node_modules/.bin/orval --config orval.config.ts --project <name>`, then run prettier on the generated file or the diff is huge.
