---
name: reference-develop-merge-deletes-source-branch-repush-for-main
description: "After an MR into develop merges, GitLab deletes the source branch; the git-flow MR into main then fails 'source_branch does not exist' — re-push the same merged head SHA under the branch name"
metadata:
  type: reference
---

Seen 2026-11-03 on quota-management-backend, bola-backend and provider-management-frontend: `glab mr create --target-branch main` → 400 `source_branch: does not exist` right after the develop MR merged.

**How to apply:** `git push origin "${SHA}:refs/heads/${BRANCH}"` with the pre-merge head (check `git merge-base --is-ancestor $SHA origin/develop` first), then open the main MR. Braces matter in zsh ([[reference-zsh-colon-modifier-eats-git-refspec]]).
