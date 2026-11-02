---
name: reference-reverted-merge-branch-is-ancestor-reapply-by-reverting-the-revert
description: A card branch merged into a wave/main branch and then reverted is already an ancestor of main — merging it again is a silent no-op; re-apply by reverting the revert commit on a new branch from main
metadata:
  node_type: memory
  type: reference
  originSessionId: acb9d813-6c1d-48df-8dfa-e8f6f22c298b
  modified: 2026-11-02T15:24:36.224Z
---

When a card branch was merged (e.g. into `feature/MVP-wave1`) and the merge was later reverted, and that wave branch reached `main`, the card branch's head is an ancestor of `main`. `git merge-base origin/main <branch>` returns the branch head itself, and `git merge` reports nothing to do — the code never comes back, with no error.

Seen 2026-11-02 on chat-core: `feature/AI-313-lead-from-chat@7f85829` merged in `a4d6633`, reverted in `1547e57`, both on main via !191.

**How to apply:** cut a new branch from `origin/main` (`<old>-v2`), `git revert --no-edit <revert-sha>` (resolve list-append conflicts such as `migrations.go`), reword that commit, then rework on top. Check with `git merge-base --is-ancestor <old-head> origin/main` before assuming a merge will bring anything. Related: [[reference_rtk_git_log_hides_merge_commits]].
