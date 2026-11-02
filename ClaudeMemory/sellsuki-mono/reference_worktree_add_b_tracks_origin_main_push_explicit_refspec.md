---
name: reference_worktree_add_b_tracks_origin_main_push_explicit_refspec
description: "git worktree add -b <branch> <path> origin/main sets the new branch's upstream to origin/main — a bare `git push` then targets main (refused by push.default=simple, silently with -q). Always push with an explicit refspec"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2ed39e79-b1a0-4aac-9faa-c61a3445c73a
  modified: 2026-11-02T11:46:30.731Z
---

Seen 2026-11-02 (AI-360, kit): `git push -q` from a branch created by `git worktree add -b feature/X <path> origin/main`
printed nothing and pushed nothing — the upstream was `origin/main`, so simple mode refused the name mismatch. Had
push.default been `upstream`, it would have pushed onto main.

**How to apply:** push with `git push -u origin HEAD:refs/heads/<branch>` (or `--set-upstream-to=origin/<branch>`
right after the first push), then verify with `git ls-remote origin <branch>` and `git ls-remote origin main`.
Related: [[reference_monorepo_worktree_from_main_lacks_ai_submodules_and_tooling]].
