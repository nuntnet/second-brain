---
name: reference_local_stack_add_conflict_means_stale_stack_simulate_rebuild
description: "`local-stack.sh add` conflicting on files your branch never touched = that repo's local/stack is built on an old base; simulate a rebuild in a scratch worktree (every listed branch in main? stack-only commits?) before `rebuild`"
metadata:
  type: reference
---

Seen 2026-11-14 (AI-49 reopen+drag): `scripts/local-stack.sh add backend/sellsuki-chat-core <branch>` conflicted on 16 files,
including `cmd/chat_core_server/main.go` and migrations my branch never touched. Cause: chat-core `local/stack` was 99
commits behind `origin/main` (built before the 11-11 wave merged), so a branch cut from main could not merge into it.

Safe sequence that worked:
1. `git worktree add --detach $TMPDIR/sim origin/main`, then for each branch in `.git/local-stack/manifest` for that repo:
   `merge-base --is-ancestor` (already in main?) else try a merge; finally merge yours. All 20 were already in main.
2. `git rev-list --no-merges origin/main..local/stack` + `git cherry` to find commits made straight onto local/stack
   (there were 4 — the /end work cherry-picked by hand; their reviewed versions were in main under other SHAs).
3. Check migrations main adds vs the local DB (`docker exec sellsuki_mono-postgres-1 psql -U postgres -d chat_core`).
4. Keep a backup ref (`git branch local/stack-before-<x> local/stack`), then `local-stack.sh rebuild <repo>` and `add`.
5. Confirm the running binary is newer than the stack's last reflog entry (`ls -l tmp/main`, `git reflog local/stack`) —
   a 401 from the session middleware cannot tell you whether a new route exists.

Related: [[reference_local_stack_branch_script]], [[reference_local_stack_stale_branches_per_repo]]
