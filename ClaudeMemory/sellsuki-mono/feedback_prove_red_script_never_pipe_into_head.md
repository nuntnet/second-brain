---
name: feedback_prove_red_script_never_pipe_into_head
description: "A mutate → test → git checkout prove-red script piped into `head` dies of SIGPIPE before its restore step and leaves the mutation in the tree (and git checkout also wipes any uncommitted edit to that file); commit before mutating, never truncate its output"
metadata:
  type: feedback
---

Seen 2026-10-29 on AI-334 (chat-core). A helper `pr.sh` changed one value, ran `go test`, then `git checkout -- <file>` to restore. Two failures:
1. `pr.sh … | head -4` — head closed the pipe, the script was killed before `git checkout`, and the mutation (`TypeCollectContact` → `TypeCreateOrder`) stayed in the worktree while a verifier subagent and a full suite were using it.
2. `git checkout -- <file>` as the restore also throws away any uncommitted edit made to that file before the mutation.

**Why:** a leftover mutation silently poisons every concurrent run (verifier, full suite) on the same worktree, and its verdict.
**How to apply:** commit before each mutation round; never pipe the prove-red script into `head`/`grep -m` (filter inside the script instead); after the round run `git status --short` and `git diff` before anything else. Also: Go in this sandbox fails with "package errors is not in std" — that is the toolchain being unreadable, run `go` unsandboxed. Related: [[reference_chat_core_db_tests_need_p1]].
