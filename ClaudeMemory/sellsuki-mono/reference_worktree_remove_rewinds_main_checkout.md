---
name: reference_worktree_remove_rewinds_main_checkout
description: "git worktree add -f on a branch already checked out in the main repo, then removing that worktree, rewinds the main checkout's FILES while HEAD stays on the new commit — looks like uncommitted work, is actually a stale revert"
metadata:
  node_type: memory
  type: reference
---

Hit 2026-09-04 in `backend/sellsuki-ai-agent`.

The submodule checkout was on `docs/AI-82-env-contract`. I ran
`git worktree add -f <tmp> docs/AI-82-env-contract` — `-f` overrides git's normal
refusal to check out one branch twice — committed and pushed from the worktree,
then `git worktree remove --force <tmp>`.

Result in the MAIN checkout:

```
HEAD           = 27b6084   (the pushed commit, correct)
index + files  = 9cc9ed2   (the state BEFORE it)
git status     = 4 files staged-modified
```

It reads exactly like uncommitted work in progress. It is the opposite: a diff
that **removes** commits already on the remote. Committing it would have
reverted the push; `git stash`-ing it would have hidden that.

**Check before acting:** `git diff --cached --stat`. If every hunk deletes work
that `git log origin/<branch>` already shows, the working tree is stale, and the
fix is `git restore --source=HEAD --staged --worktree .` (scoped, and not
`reset --hard`, which the monorepo rules discourage).

**Avoid it:** don't `worktree add -f` a branch the main repo already has checked
out. Either branch from `origin/<branch>` under a new name in the worktree, or
move the main checkout to `main` first.

Also seen the same day: the removal left a **prunable** stale entry pointing at
a deleted `/private/tmp/...` path — `git worktree prune` clears it, and
`git worktree list` is how you notice.
