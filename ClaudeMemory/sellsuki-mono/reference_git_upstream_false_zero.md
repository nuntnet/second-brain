---
name: reference-git-upstream-false-zero
description: git log @{u}..HEAD reports 0 unpushed commits when the branch has no upstream — compare against origin/<branch> explicitly
metadata:
  type: reference
---

Several branches in this workspace have **no upstream configured** — notably the monorepo's `chore/ai-mvp-local-run`.

On such a branch, `git log --oneline @{u}..HEAD 2>/dev/null | wc -l` prints **0**: `@{u}` fails, stderr is swallowed by the redirect, and an empty stdout counts as zero. It reads as "everything is pushed" when nothing is.

2026-08-18 this made me report the monorepo as fully pushed. Comparing explicitly showed **7 unpushed commits**.

**Always** resolve the remote ref by name instead:

```
git fetch origin <branch>
git log --oneline origin/<branch>..HEAD
```

Same class of bug as any `2>/dev/null` on a check whose failure mode is silence — the guard reports success because it could not run.

Related: [[reference-submodules-are-shallow-clones]], [[feedback-verify-absence-claims]]
