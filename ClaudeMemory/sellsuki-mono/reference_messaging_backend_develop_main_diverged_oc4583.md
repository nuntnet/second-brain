---
name: reference_messaging_backend_develop_main_diverged_oc4583
description: "sellsuki-messaging-backend develop and main have diverged: any branch cut from main conflicts when merged into develop (5 OC-4583 email files). Develop last moved 2026-10-08, and AI-216/AI-326/AI-298 messaging work went straight to main. Never resolve these conflicts in the GitLab UI."
metadata:
  node_type: memory
  type: reference
  originSessionId: 3aac068a-3349-436b-b707-7d3b887a3e29
  modified: 2026-10-29T00:12:28.737Z
---

Found 2026-10-28 when opening messaging-backend MR !84 (AI-216 webhook fix) into `develop`. GitLab showed `detailed_merge_status=conflict`.

- **Divergence:** 24 commits on each side, counted from merge-base `8026926e`. That is 14 non-merge commits only on `main` and 7 only on `develop`. `develop`'s tip `55269bd` is from 2026-10-08.
- **Cause:** OC-4583 (member invite email/SMS) landed one way on `main` (`0ef610d`, `f7a207e`, `cb04768`) and another way on `develop` (`c64045e`, `55269bd`). `develop` also holds messaging routing work (`0fd07be`, `0741428`, `8b33b97`) that `main` lacks.
- **Conflicting files:** `docs/provisioning/oc4583-member-invite-actions.md`, `src/interface/fiber_server/route/email/{route.go,spec.gen.go}`, `src/use_case/email.go` and `src/use_case/email_test.go`. Any branch cut from `main` hits them, whatever it touches.
- **What actually happens:** the AI chat messaging MRs (AI-216's !80 and !82, plus AI-326 and AI-298) went straight to `main`, and `develop` was left untouched. See [[reference_messaging_chat_module_staging_flip_2026_10_27]].

**How to apply:**
- Before opening a messaging MR into `develop`, run `git merge-tree --write-tree --name-only origin/develop origin/<branch>`.
- If it conflicts, tell the user before opening, and let them choose between targeting `main` directly and reconciling `develop` first (that needs the OC-4583 owner).
- **Never press "Resolve conflicts" in the GitLab UI.** It merges `develop` into the feature branch, and the same branch later carries `develop`'s unreleased work into `main` (Git flow rule 4).
- Against `develop`, the Changes tab shows every one of `main`'s 24 extra commits, so name the single commit to review.

**2026-10-29 (AI-333): nunt chose "Fix develop first" over targeting `main` directly.** When asked whether a messaging MR should skip `develop`, the answer was to reconcile `develop` first and keep the git flow. Don't default to `main`; ask, and expect this answer. The owner of the conflicting side is **nithiphat** (OC-4583). The same feature exists twice: `fa814b6` on main and `c64045e` on develop. After that, main got `35bb85f` `0ef610d` `f7a207e` `cb04768` (10-09) and develop got `55269bd` (10-06). The fix is to merge `main` into `develop`, with nithiphat resolving the 5 email files. That one step unblocks !84 (AI-216) and AI-333 together. Ledger: `docs/cards/AI-333.md`.

**The reconcile step is MR !85** (`fix/AI-333-redact-chat-5xx-into-develop` `c32b9f6`, cut from `develop` `ad80a56`, opened 10-29 01:36). It takes main's side for all 5 files and brings all 27 of main's commits into `develop`. nithiphat still needs to confirm that `X-Wait-For-Provider` may reach `develop`. On 10-29 nunt also chose to keep !84 on `develop`, not retarget it. A trial merge of !84's `6a5156c` on top of `c32b9f6` is clean, so once !85 merges, !84 merges with no further work. Recheck with `git merge-tree` against the new `develop` tip before merging.

Same family as [[reference_oc2plus_backoffice_develop_main_diverged_2026_10_11]] and [[reference_rps_and_management_backend_develop_main_diverged]].
