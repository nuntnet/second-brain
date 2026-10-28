---
name: reference_sandbox_blocks_submodule_claude_commands_skills_and_worktree_add
description: "Bash sandbox denies writes to any repo's .claude/commands and .claude/skills (submodules too), and `git worktree add` for a submodule fails half-way once cwd has left the monorepo root"
metadata:
  node_type: memory
  type: reference
  originSessionId: a7da547e-75b9-4bcd-99f8-99b884a2e2d5
  modified: 2026-10-28T18:31:21.252Z
---

Seen on AI-348 (2026-10-29):

- **`.claude/commands/*` and `.claude/skills/*` are write-protected inside the sandbox in every repo, submodules included.** You get `PermissionError: Operation not permitted`. Writes to `.claude/rules` and `.claude/knowledge` work. Don't bypass it; list the stale files in the card ledger as a follow-up.
- **`git worktree add` from a submodule failed with "Could not reset index file to revision 'HEAD'"** after the session's cwd had moved into the submodule. Git writes to `/Users/nunt/sellsuki_mono/.git/modules/...`, which was then outside the writable cwd. The branch had been created but there was no checkout. Fix: `git worktree prune`, then `git worktree add <path> <existing-branch>` unsandboxed. `git worktree remove` needs the same.
- The submodule's own `.venv` works from a worktree: `/Users/nunt/sellsuki_mono/backend/rag-core/.venv/bin/python -m pytest …`. `uv run` fails because `~/.cache/uv` is not writable.

Related: [[reference_sandbox_blocks_go_cli_tls_and_ssh]] (glab GETs need unsandboxed too) · [[reference_auto_mode_blocks_glab_mr_create]].
