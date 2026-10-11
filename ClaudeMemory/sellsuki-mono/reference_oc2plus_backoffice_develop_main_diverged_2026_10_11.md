---
name: reference-oc2plus-backoffice-develop-main-diverged-2026-10-11
description: "Measured 2026-10-11: ALL FIVE OC2Plus repos — backoffice-api 44 conflicts (develop 299/105 vs main), backoffice FE 22 (184/106), 3rdparty-api 25 (91/31), member-api 40 (144/96), member FE 14 (101/85) — origin/main itself does not merge into develop, so git-flow step 2 (main-cut branch → develop) is impossible; OC-3559 used 'develop twins'; OC-4734 MRs !720/!714 sit on develop unmergeable awaiting the PO's call"
metadata:
  node_type: memory
  type: reference
  originSessionId: 5077123b-d938-4a36-b54b-335561e7b9a9
  modified: 2026-10-11T09:07:33.606Z
---

`git merge-tree --write-tree origin/develop origin/main` on 2026-10-11:

| repo | main→develop conflicts | develop ahead / behind main |
|---|---|---|
| backend/oc2plus-line-crm-service-backoffice-api | 44 (incl. `spec/v1.yaml`, `go.mod`) | 299 / 105 |
| frontend/oc2plus-linecrm-frontend-backoffice | 22 | 184 / 106 |
| backend/oc2plus-line-crm-service-3rdparty-api | 25 | 91 / 31 |
| backend/oc2plus-line-crm-service-member-api | 40 | 144 / 96 |
| frontend/oc2plus-linecrm-frontend-member | 14 | 101 / 85 |

A branch cut from `main` carries main's divergence, so it conflicts the same way
(44 / 24 for OC-4734's branches). Nothing cut from main can reach the dev cluster
through `develop` in these two repos until someone reconciles them. The house
workaround (OC-3559, 2026-10-05) is a **develop twin**: the same change cherry-picked
onto a branch cut from `develop`, MR'd into develop; the main-cut branch goes to
main after dev passes. For a spec-level test this also means a different
baseline: the display-label ratchet is red on develop's `v1.yaml` for 6 fields
the main spec does not have.

**Update 2026-10-11 18:22: backoffice-api !722 and backoffice FE !718 (OC-4736) were merged — those two repos are reconciled (main→develop = 0 conflicts). 3rdparty-api !304, member-api !223, member FE !154 still open. Lesson: a commit pushed to an MR branch after the MR merged never reaches the target — re-read MR state before pushing and before reporting it.**

**Decision 2026-10-11 (PO): reconcile once** — card OC-4736 merges main into develop via `refactor/OC-4736-sync-main-into-develop` per repo (backoffice-api + FE first; the other three await the PO's call).

**How to apply:** before promising "merge to develop to test on dev" for these
repos, run the merge-tree check; if it conflicts, say "develop twin" in the card
ledger and the MR report, and ask the PO whether reconciling develop↔main is the
real card. Retargeting an MR via `glab api` was refused by the tool permission
gate in auto mode on 2026-10-11 — a human does it or grants it.
Related: [[feedback-git-flow-main-develop-main]], [[reference-glab-auto-merge-merges-immediately-without-ci-gate]].
