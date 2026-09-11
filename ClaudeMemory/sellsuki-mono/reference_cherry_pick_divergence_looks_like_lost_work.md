---
name: reference_cherry_pick_divergence_looks_like_lost_work
description: "\"Another team's work disappeared\" in CCS/CCS3 was main-vs-develop divergence, not a clobber — how to prove it in five checks, and the cherry-pick habit that causes it"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-11T05:14:32.176Z
---

2026-09-11: a team reported their work (named: slip2go, "and other things") had vanished from `sellsuki-central-control-backend` and `sellsuki-company-management-frontend`, suspected cause "we merged without pulling". **Nothing was lost.** Five checks settled it, in this order — run them before touching anything:

1. **Is it actually absent?** `git grep -il <feature> origin/main` AND `origin/develop` — not `HEAD`. slip2go was present on **both** FE branches (17 files) and `git diff origin/main origin/develop -- <paths>` was **empty**, last touched by another author months earlier. (The local checkout said 0 files because it sat on a stale unpushed branch 284 commits behind — the [[reference_grep_stale_branch_not_origin]] trap, from the reporting side this time.)
2. **Could a push have destroyed it?** `glab api projects/<enc>/protected_branches` → `main` had `allow_force_push=false` in both repos ⇒ **commits on main cannot be lost**, so any "missing from main" is a promotion gap, never a clobber. (`develop` was unprotected in both — that is where a force push *could* bite.)
3. **Did any merge delete the target's content?** For the last ~12 merges on main: `git diff --diff-filter=D --name-only M^1 M` and count deletion-only files in `git diff --numstat M^1 M`. All zero in both repos.
4. **Is the history even complete?** `test -f "$(git rev-parse --git-dir)/shallow"` — both were full. A shallow submodule ([[reference_submodules_are_shallow_clones]]) makes "it never existed in history" unprovable, so check this *before* claiming absence.
5. **Is it on the other branch?** `git rev-list --count origin/main..origin/develop` then `git log origin/main..origin/develop --no-merges`. This is where the real finding was.

**What was actually true:**
- **slip2go's backend half was never built.** `grep slip2go` across every branch and all history of CCS backend = 0. The FE calls `POST /company/{id}/slip2go/oauth/callback`, and its own comment calls that a *proposed* contract (PAT-2634). "Feature gone" meant "feature never worked end to end".
- **MS-1345 (another author) was on `develop`, not `main`** — `variant_option` 7 hits on develop, 0 on main. CCS backend and CCS3 FE both deploy **main → staging**, so anyone checking staging sees develop-only work as missing.
- Timing looked damning and wasn't: the develop→main merge landed 5 minutes *after* their commit, but an MR merges the sha recorded when it was created, not `develop` as of the click.

**🔴 The habit that caused the confusion — mine.** Instead of merging one branch into the other, the same fix was applied separately to `main` and `develop`, leaving same-subject/different-hash pairs (4 of them: point-claim permission ×2, BOLA_API_BASE_URL, BOLA_SYSTEM_TOKEN). Each side then reports the other's commit as "not present", `git log A..B` is noise in both directions, and the next real merge conflicts against your own duplicate. **How to apply:** land a change once, on one branch, then merge that branch into the other and let the merge carry it. If a hotfix genuinely must go to both, cherry-pick with `-x` so the message records the origin, and say so in the MR.

**How to answer the question the fast way next time:** "is X on main?" = `git merge-base --is-ancestor <sha> origin/main`. "is the *content* on main?" = `git grep -c <symbol> origin/main` — a cherry-pick means the sha answer and the content answer differ, and only the content answer matters to the person asking.
