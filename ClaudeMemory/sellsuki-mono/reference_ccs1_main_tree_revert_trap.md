---
name: reference-ccs1-main-tree-revert-trap
description: "CCS1 (sellsuki-system-management-frontend) main was tree-reverted to the old UI on 2026-09-29 (AKI-4098) — merging main→develop wipes the new UI, develop→main yields a hybrid; bringing the new UI back needs revert-the-revert"
metadata:
  node_type: memory
  type: reference
  originSessionId: 579ce85e-22c1-442b-b02b-216071392e6b
  modified: 2026-09-29T16:40:20.103Z
---

On 2026-09-29 `frontend/sellsuki-system-management-frontend` main got !81 (AKI-4098,
nitchanan.kati): commit `4c25e74` "revert: restore tree to 87b9a08 (keep CI)" — the
whole tree back to the pre-`feat/ui-migration` UI of ~6 months earlier — plus 4 fixes
on top (real user in sidebar header, 404 page, shipmunk field names, cleanups).
main deploys staging/prod; develop (new DS UI, 26 commits mostly kimzey, + PAT-2735)
deploys dev. `origin/main-2` = main just before the revert (= merge-base `e4a7fb5`);
`origin/recovery-main-th` is an older ancestor of both.

**Why it is a trap:** a tree-revert makes git believe the new-UI changes were undone
on purpose.
- Merging **main → develop** (or into any branch cut from develop) applies the revert
  and silently deletes the new UI from that branch.
- Merging **develop → main** does NOT bring the new UI back: files unchanged on develop
  since the merge-base stay reverted; only the 29 develop-only commits land → a hybrid
  that matches neither line.
- To return the new UI to main: first revert `4c25e74` (revert-the-revert), then merge.

**How to apply:** CCS1 work targets develop only (user choice B, 2026-09-29) until the
new UI is re-released; never "merge origin/main into" a CCS1 branch; say so in any
CCS1 MR. Related: [[reference-central-config-develop-lacks-main-seeds]],
[[feedback-no-develop-to-main-promotion-mrs]].
