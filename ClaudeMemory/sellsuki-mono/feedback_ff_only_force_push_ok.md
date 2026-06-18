---
name: feedback-ff-only-force-push-ok
description: PIS GitLab repos are fast-forward-only → rebasing+force-pushing FEATURE branches is allowed
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9e057444-0352-41d3-9463-fb0dd304ba9a
---

The monorepo CLAUDE.md says "NEVER force push to any branch." On 2026-06-18 the user clarified an
exception: PIS GitLab projects (`pis-api`, `pis-frontend`, likely others) use **`merge_method: ff`
(fast-forward only)**. An FF-only MR can't merge unless the source branch is rebased onto target, and
rebasing an already-pushed branch requires a force push.

**Why:** the "never force push" rule is really about protecting shared/protected history (main). For
the team's own FF-only repos, rebasing + `git push --force-with-lease` a personal FEATURE branch is
the required, normal workflow to clear `conflict` / `need_rebase` merge blocks.

**How to apply:**
- main / shared branches: still NEVER force push, NEVER rewrite pushed history.
- feature branches in FF-only repos: rebasing + `--force-with-lease` is OK (user approved). Prefer
  `--force-with-lease` over `--force`.
- For a `need_rebase` block with NO conflicts, prefer GitLab's server-side rebase
  (`glab api -X PUT .../merge_requests/<iid>/rebase`) — no local force-push needed.
- For a `conflict` block, rebase locally, resolve, build+test, then `--force-with-lease`.
- Check the block reason first: `glab api projects/<urlencoded-path>/merge_requests/<iid>` →
  `detailed_merge_status`, `has_conflicts`, `merge_method`. See [[project_pis_frontend_local_testing]].
