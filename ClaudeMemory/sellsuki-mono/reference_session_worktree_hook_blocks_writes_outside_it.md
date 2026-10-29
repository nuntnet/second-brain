---
name: reference_session_worktree_hook_blocks_writes_outside_it
description: "In a Claude worktree session, a PreToolUse hook refuses Write/Edit anywhere outside .claude/worktrees/<name>/ — put submodule worktrees (git worktree add) INSIDE the session worktree, e.g. <session-wt>/wt/<branch>"
metadata:
  node_type: memory
  type: reference
  originSessionId: c4c0d24b-4441-439e-8950-fa80736ecd27
  modified: 2026-10-29T09:38:08.284Z
---

Seen 2026-10-29 (AI-346): a chat-core worktree created at `/Users/nunt/sellsuki_mono/.worktrees/AI-346-chat-core` could
not be edited — the Write hook said the path "is in the base repo checkout" and pointed at the session worktree instead.
Bash/python writes are not blocked, but the Write/Edit tools are.

**How to apply:** create submodule worktrees under the session worktree from the start
(`git -C backend/<repo> worktree add <session-wt>/wt/fix <branch>`, unsandboxed), or `git worktree move` an existing one
there. Same for a docs worktree of the monorepo (`<session-wt>/.docs-<CARD>`). Remove them before the turn ends.
Related: [[reference_monorepo_worktree_from_main_lacks_ai_submodules_and_tooling]],
[[reference_sandbox_blocks_submodule_claude_commands_skills_and_worktree_add]].
