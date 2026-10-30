---
name: parallel-bash-cd-and-zsh-wordsplit-use-git-dash-c
description: "Two Bash traps in this workspace — parallel tool calls share a cwd so `cd worktree && git checkout -b` can run in the shared root checkout; zsh does not word-split $VAR lists. Use `git -C <abs>` and `bash -c` for loops."
metadata:
  node_type: memory
  type: reference
  originSessionId: 0ac30775-44de-4151-a526-9e00752dc658
  modified: 2026-10-30T13:24:41.127Z
---

On 2026-10-30 a `git checkout -b scratch/...` meant for a worktree ran in the shared root checkout (`/Users/nunt/sellsuki_mono`, on `fix/AI-49-safe-worker-activation`). Cause: two Bash calls issued in the same message share one shell cwd, and the `cd` in the other call (the Decision Inbox command) changed it between them. Another session then committed onto my scratch branch, believing it was the AI-49 branch. Repaired by `git checkout fix/AI-49-safe-worker-activation && git merge --ff-only scratch/... && git branch -d scratch/...` — their commit was kept, nothing lost.

**Why:** the root checkout is shared by every session and other sessions commit docs there constantly (`repos.md`: never switch branches there).

**How to apply:**
- In any worktree work, never rely on `cd`. Use `git -C /abs/worktree <cmd>` for every git call, and never put a branch-changing command (`checkout`, `switch`, `reset`) in a message that also holds another call that runs `cd`.
- zsh does not word-split an unquoted `$LIST`: `for c in $OWN` iterates once over the whole string, and `git log $OWN` fails with "ambiguous argument". Run loops under `bash -c '...'`.
- Do not use `"$p"` pairs like `set -- $p` in zsh either; call the commands plainly.
- Related: [[reference_tmpdir_differs_sandboxed_vs_unsandboxed_bash]] (logs written by an unsandboxed job are invisible to a sandboxed wait loop), [[reference_session_worktree_hook_blocks_writes_outside_it]].
