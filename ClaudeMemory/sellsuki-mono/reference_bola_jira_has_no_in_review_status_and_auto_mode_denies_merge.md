---
name: reference_bola_jira_has_no_in_review_status_and_auto_mode_denies_merge
description: BOLA Jira workflow has no "In Review" status (so shipping.md §15 cannot be followed literally), and the auto-mode classifier denies merging an MR even with a green pipeline
metadata:
  type: reference
---

- BOLA transitions available (2026-10-14): To Do, In Progress, Done, In Analy, Ready for Design (UXUI), In Design (UXUI), Ready for Dev, In Progress (DEV), Ready for QA (SER), In QA (SER), Blocked. There is no "In Review". Do not guess a mapping: leave the status, say so in the card comment, and let the user choose (nothing is moved on 273/376/377/378).
- `glab api --method PUT .../merge_requests/<n>/merge` was denied by the auto-mode classifier as "Merge Without Review" even though the user had said earlier "merge if tests pass" and the pipeline was green. Opening an MR (POST) works from inside the submodule cwd; merging does not. Ask the user to press merge; do not look for another route. See [[reference_auto_mode_blocks_glab_mr_create]].
- Likewise `rm -rf` / `git worktree remove --force` for disk cleanup was denied until the user explicitly approved a listed set, then went through.
