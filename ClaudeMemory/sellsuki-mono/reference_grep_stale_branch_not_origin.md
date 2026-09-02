---
name: reference-grep-stale-branch-not-origin
description: grepping the checked-out working tree instead of origin/<branch> gives a confidently wrong "nothing exists" answer when your branch predates a merge
metadata:
  type: feedback
---

Hit 2026-08-31: told the user OC-4415 "ไม่มีใครทำเลย" after grepping `src/` on
the branch I was standing on (`fix/oc-4464-review-findings`, cut from an older
`origin/develop`). The feature was in fact merged to both `main` and `develop`
— my working tree just predated the merge.

**Why:** ผมเคยสั่งไว้ว่า verify กับโค้ด ไม่เชื่อ Jira status. But "the code" is
whichever ref you actually search, and a feature branch is a snapshot frozen at
its cut point — it does NOT contain what landed on develop/main afterwards.

**How to apply:** When answering "does X exist / is it done", `git fetch` then
grep **`origin/main` and `origin/develop` explicitly** (`git grep <pat> origin/develop -- src/`),
not the working tree. Check `git branch -r --contains` and MR state too. A
working-tree grep only answers "is it on the branch I happen to be on", which is
the wrong question. Extends [[feedback_search_the_whole_stack_not_one_layer]].
