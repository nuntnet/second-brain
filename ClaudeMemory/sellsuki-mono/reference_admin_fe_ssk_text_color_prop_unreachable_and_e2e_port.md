---
name: reference_admin_fe_ssk_text_color_prop_unreachable_and_e2e_port
description: "ai-chat-admin console — ssk-text `color=` prop is set inside the shadow root (app CSS cannot override it short of a DS token override); parallel Playwright runs collide on :5196; a CI e2e shard printing TTTT is a starved runner"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9d306d0f-f9b8-4442-8af9-75e8c882d25e
  modified: 2026-10-30T10:21:23.004Z
---

Seen 2026-10-30 landing AI-309/AI-310 (MR !113/!114, `frontend/ai-chat-admin-frontend`):

1. **`<SskText color="background.600">` cannot be recoloured from app CSS.** The DS `p` in the shadow root sets `--color: var(--ssk-colors-<color>-<shade>…)` on itself and `color: var(--color)`. A rule on the host (`.x > ssk-text { color: … }`) changes the host only. The only outside lever is overriding `--ssk-colors-background-600` on the host, which DESIGN.md D7 forbids. So when a background changes under an ssk-text with an explicit `color`, the fix is the TSX prop (drop it to inherit), and any card AC that says "CSS only" conflicts. Without a `color` prop, ssk-text inherits the host colour.
2. **Parallel Playwright runs (you + a verifier) collide on :5196** (`reuseExistingServer: false` → "already used", 0 tests run, exit looks clean in JSON). Use a copy of the config: `sed 's/5196/5200/g' playwright.config.ts > zz-pw.config.ts` and `-c zz-pw.config.ts`; give each agent its own port.
3. **A CI e2e shard whose trace shows `·TTFTTTT…` for most tests while the other shards of the same commit passed is a starved runner**, not the code. Cancel and retry that job by id once (`glab api -X POST projects/<path>/jobs/<id>/cancel`, then `/retry`).
4. Local unit timeouts (~6.5 s) in InboxPage.paging / GoogleDriveImports tests appear when Playwright runs at the same time; re-run alone before calling them failures.

See [[reference_admin_fe_testid_shifts_axe_baseline_and_turbo_loose_env]], [[reference_admin_fe_worktree_borrowed_node_modules_test_main_core]].
