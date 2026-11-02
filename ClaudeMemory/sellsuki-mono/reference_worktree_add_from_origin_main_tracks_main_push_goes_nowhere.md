---
name: reference_worktree_add_from_origin_main_tracks_main_push_goes_nowhere
description: "`git worktree add -b feature/X <path> origin/main` sets the new branch's upstream to origin/main, so a bare `git push` pushes nothing (push.default=simple, names differ) and rtk still prints `ok ✓` — push with `HEAD:refs/heads/<branch>` and confirm with ls-remote"
metadata:
  type: reference
---

Seen 2026-11-02 (AI-370, console worktree): the first `git push -u origin feature/AI-370-shell-and-shared` created
the remote branch, but `git worktree add -b feature/AI-370-shell-and-shared <path> origin/main` had already set the
upstream to `origin/main`, and later bare `git push` calls did nothing. rtk printed `ok ✓` each time, so five
commits looked pushed and were not; `git ls-remote` showed the first commit only.

**How to apply:**
- Create submodule worktrees with `git worktree add --no-track -b <branch> <path> origin/main`.
- Push with `git push origin HEAD:refs/heads/<branch>`, then `git ls-remote origin <branch>` must equal `git rev-parse HEAD`.
- To repair: `git branch --set-upstream-to=origin/<branch>` after the explicit push.

Same family as [[reference-git-upstream-false-zero]] (a check that reports success because it could not run).
