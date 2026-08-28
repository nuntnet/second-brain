---
name: reference-bak-restore-drops-comments
description: restoring a file from a .bak taken mid-edit silently reverts comment blocks; the diff hides it inside an unrelated feature commit
metadata:
  type: reference
---

Mutation testing by `cp file /tmp/f.bak` → mutate → `cp /tmp/f.bak file` is safe
only if the `.bak` was taken **after** every edit in that turn. Take it earlier
and the restore silently reverts whatever came between — in practice the doc
comments, because those are what a rewrite touches most.

Cost seen 2026-08-28 (OC-4464): commit `c73610b`, whose message said only "add
audit events", deleted **101 comment lines** from `point_claim_ocr.go`,
including the one 🔴 D-D1 warning stating the feature must never change a
claim's status. Build green, tests green, no signal at all. Found later only by
reading the diff during [[feedback_list_open_mrs_before_opening_one]]-style
review.

**Before committing after any mutation run: `git diff --stat` and skim
`git show <staged> | grep '^-'` for deletions you did not intend.** A commit
that adds a feature should not have a net-negative comment count.

Related trap in the same session: `git checkout <file>` to undo a mutation
**deletes an unstaged new test file** written earlier in the same turn — it
restores from the index, which never had it. Stage new files before mutating.
