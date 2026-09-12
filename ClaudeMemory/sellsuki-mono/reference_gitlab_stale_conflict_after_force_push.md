---
name: reference_gitlab_stale_conflict_after_force_push
description: "GitLab keeps reporting detailed_merge_status: conflict / has_conflicts: true on an MR that is a clean descendant of its target after a force-push — a stale cached merge ref, cleared by GETting the MR's /merge_ref endpoint"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 6266b423-4293-4770-9853-0c294845f245
  modified: 2026-09-12T07:56:25.067Z
---

After force-pushing a rebased branch, `glab api projects/:id/merge_requests/<iid>`
can keep answering:

```
merge_status: cannot_be_merged
detailed_merge_status: conflict
has_conflicts: True
merge_error: None
```

**even when the branch is a strict descendant of the target** — verified locally with
`git rev-list --count origin/develop..origin/<branch>` = N and the reverse = **0**, which
is a fast-forward and cannot conflict. Waiting does not clear it; it survived a full
pipeline run (~4 min) on both MRs it hit.

`merge_error: None` is the tell: a real conflict usually names one.

**The fix — ask GitLab for the merge ref, which forces it to recompute:**

```bash
glab api "projects/:id/merge_requests/<iid>/merge_ref"   # -> {"commit_id":"…"}
```

A `commit_id` coming back *is* the proof there is no conflict (GitLab just merged it
to produce that ref). Re-read the MR immediately after and it flips to
`detailed_merge_status: mergeable`, `has_conflicts: false`. Both times, one call was
enough and the merge then went through with no `--force` of any kind.

Seen 2026-09-12 on `oc2plus-linecrm-frontend-member` MRs !69 and !67, both after a
`push --force-with-lease` of a freshly rebased branch (project `merge_method: merge`,
`only_allow_merge_if_pipeline_succeeds: true`).

**Do not react to the stale flag by rebasing again** — the second rebase finds nothing
to do, the force-push re-arms the same stale state, and you lose another pipeline cycle.
Check ancestry locally first; if the branch is ahead-N/behind-0, the flag is wrong.

Related: [[reference_glab_ci_status_stale_pipeline]] (same family — glab reporting a
cached answer), [[feedback_ff_only_force_push_ok]].
