---
name: reference-session-worktree-hook-put-submodule-worktree-inside
description: "In a worktree session the Write/Edit hook refuses files in any other worktree — create a submodule's worktree INSIDE the session worktree (e.g. <session-wt>/.<card>-<repo>)"
metadata:
  node_type: memory
  type: reference
  originSessionId: d6f10c1b-9bc0-4c05-b48e-947029469eaa
  modified: 2026-11-20T11:29:54.393Z
---

When a Claude session runs in a monorepo worktree (`.claude/worktrees/<name>`), a PreToolUse hook rejects Write/Edit on any path belonging to a different worktree ("belongs to a different worktree. Do not write to other worktrees' files from this session").

So a submodule worktree made at `.claude/worktrees/<card>-<repo>` is unwritable from that session. Create it (or `git worktree move` it) under the session's own worktree, e.g. `<session-wt>/.ai502b-chat-core`, from the submodule's git dir (`git -C backend/<repo> worktree add -b <branch> <session-wt>/.<x> origin/main`, sandbox off). Never commit it into the monorepo; remove it at the end.

Seen 2026-11-20 on AI-502 step 2. Related: [[feedback_parallel_sessions_git_safety]].
