---
name: reference_prepush_credential_hook_installed
description: "A gstack pre-push credential guard is installed in 56 repos here (monorepo root + every submodule) as of 2026-09-17 — it blocks pushes containing HIGH-severity secrets; if a push is refused, rotate and remove the secret, never reach for --no-verify"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-17T16:03:36.022Z
---

**Installed 2026-09-17**, at the user's request, via
`~/.claude/skills/gstack/bin/gstack-redact install-prepush-hook` run in the
monorepo root and each submodule. 55 submodules + root = **56 repos**; one
submodule was skipped because it is not initialised.

`gstack-config set redact_prepush_hook true` is also set, but **the flag alone
installs nothing** — it only makes `/ship` install the hook in repos it ships
from, and work here pushes with `glab`/git directly. The per-repo install above
is what actually made it live.

## What it does

Scans the **added lines of the range being pushed** and exits non-zero on a HIGH
finding (GitLab/GitHub tokens, PEM private keys, etc.). MEDIUM warns only.

Verified end-to-end on 2026-09-17 in a throwaway repo: a push carrying a
`glpat-…`, a `ghp_…` and a PEM block was blocked.

## What it does NOT do — do not treat it as coverage

- **Not enforcement.** `git push --no-verify` and `GSTACK_REDACT_PREPUSH=skip`
  bypass it. It catches accidents, not intent.
- **No history scan.** Only the range being pushed. Anything already leaked stays
  leaked — that is `/cso`'s job.
- **Skips binary, LFS and submodule files.** The root repo's hook does not see
  inside submodules, which is exactly why every submodule got its own.
- Well-known documentation example values are allowlisted on purpose
  (`AKIAIOSFODNN7EXAMPLE` scans clean). A clean scan of an obviously-fake value
  proves nothing about the hook.
- If it cannot read the diff (e.g. over its 64MB buffer) it **blocks** rather than
  passing — correct, but it means a genuinely huge diff will be refused.

## If a push gets blocked

The secret is already in a commit, so it is compromised the moment it exists in a
branch others can fetch. **Rotate it first**, then remove it from the diff with a
new commit. Never `--no-verify` past it, and never amend/force-push to hide it —
that violates the workspace git rules anyway.

Remove the hook from one repo: `gstack-redact uninstall-prepush-hook`.

Related: [[reference_harness_classifier_blocks_secrets_and_mutations]] — the
harness separately refuses to read `.env*`, so these two guards do not overlap.
