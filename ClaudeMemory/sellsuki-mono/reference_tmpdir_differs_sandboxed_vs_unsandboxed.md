---
name: reference_tmpdir_differs_sandboxed_vs_unsandboxed
description: "$TMPDIR is a different directory in sandboxed vs dangerouslyDisableSandbox Bash calls — files/worktrees made in one are \"missing\" in the other; use the absolute scratchpad path"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7fb71c5f-6e71-4687-b858-9caa035598f1
  modified: 2026-10-08T03:00:16.333Z
---

In this setup, `$TMPDIR` resolves to `/tmp/claude-501/` inside the Bash sandbox but to `/var/folders/hz/…/T/` when a
call runs with `dangerouslyDisableSandbox: true` (needed for git fetch/push, glab, and `go test` because of the go-build cache).
A worktree or file created via `$TMPDIR` in one mode is "No such file or directory" in the other (happened 2026-10-08
with a bola-backend worktree).

**How to apply:** for worktrees and files that span several calls, use the absolute session scratchpad path
(`/private/tmp/claude-501/-Users-nunt-sellsuki-mono/<session>/scratchpad/…`), never `$TMPDIR`. Related: [[reference_local_stack_branch_script]].
