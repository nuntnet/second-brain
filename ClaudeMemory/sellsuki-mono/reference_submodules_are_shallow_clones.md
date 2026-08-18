---
name: reference-submodules-are-shallow-clones
description: Submodules here are shallow clones, so merge-base/history checks silently lie — verify before concluding anything about branch ancestry
metadata:
  type: reference
---

Submodules in `sellsuki_mono` are **shallow clones** (`git rev-parse --is-shallow-repository` → `true`; there is a `.git/modules/<sub>/shallow` file).

The shallow boundary cuts off the common ancestor, so locally:

- `git merge-base A B` returns **nothing, exit 1** — no error message
- `git merge-tree` says **"refusing to merge unrelated histories"**
- `git rev-list --count <branch>` is far lower than reality (saw 47 where the MR said 189)
- root commits of two branches differ, making them look genuinely unrelated

2026-08-18 I concluded from exactly this that chat-core's MR branch and `main` had unrelated histories and that the MR could never merge normally. **That was wrong.** After `git fetch --unshallow origin` a real merge-base appeared immediately and only two files actually conflicted.

**How to not repeat it:** before drawing any conclusion about ancestry, divergence, or "can this merge", run `git rev-parse --is-shallow-repository`. If true, `git fetch --unshallow origin` first. GitLab's own `detailed_merge_status` is computed server-side on full history, so trust it over local git when the two disagree.

This is also why the CI contract gate needs `GIT_DEPTH: 0` — same root cause, different symptom.

Related: [[reference-git-upstream-false-zero]], [[feedback-verify-absence-claims]], [[reference-rtk-git-output-filtering]]
