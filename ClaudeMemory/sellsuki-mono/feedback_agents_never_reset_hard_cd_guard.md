---
name: feedback-agents-never-reset-hard-cd-guard
description: "2026-11-19 a parity-sweep subagent ran `git reset --hard` 9× in the monorepo ROOT (its `cd` into a scratch worktree failed after `shift`), wiping other sessions' uncommitted tracked edits. Briefs must forbid destructive git and require `cd X || exit`."
metadata:
  node_type: memory
  type: feedback
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-19T07:57:29.579Z
---

On 2026-11-19 14:51 a read-only "develop↔main parity" subagent looped over scratch worktrees with `cd "$W"; git reset --hard ...`. After a `shift` the `cd` failed, so every reset ran in /Users/nunt/sellsuki_mono itself. ~25 tracked root files with other sessions' uncommitted edits (card ledgers, INBOX.md, Makefile, Caddyfile, .claude/knowledge…) were reset. No Time Machine snapshot; recovery had to be rebuilt from ~/.claude/file-history + session transcripts.

**Why:** the monorepo root is shared by many sessions and Codex and is always dirty; a reset there destroys work nobody can see in git.

**How to apply:**
- Every subagent brief that touches git says: never `git reset --hard`, `checkout -- .`, `clean`, or `stash` outside a worktree you created; prefer `git worktree add --detach` + read-only commands; trial merges with `git merge-tree --write-tree` (no working tree at all).
- Any script that `cd`s into a worktree uses `cd "$W" || exit 1` and asserts `[ "$(git rev-parse --show-toplevel)" = "$W" ]` before a mutating command.
- "Read-only audit" agents get no mutating git verbs at all.

Related: [[feedback-parallel-sessions-git-safety]], [[feedback-never-pkill-pattern-kill-by-pid-on-macos]].
