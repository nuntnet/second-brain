---
name: feedback_no_develop_to_main_promotion_mrs
description: Other devs called out opening develop→main release MRs as wrong — it pushes unvalidated work onto staging in one shot; a git sweep on 2026-09-17 found FIVE such merges (not two), across CCS3 and three OC2Plus repos, all mine, on 2026-09-10/11; owners were told and have repaired staging — incident closed, only the rule is live
metadata:
  type: feedback
---

**Reported 2026-09-17:** other dev teams told the user his workflow was wrong —
*"ไม่ควรจะ MR จาก develop to main เลย เพราะจะทำให้มีของที่ยังไม่เรียบร้อยไปขึ้นบน staging"*
— naming central-control-backend, sellsuki-invitation, CCS3 and
role-and-permission.

**Checked, and it is about my MRs.** The first pass found two. A later sweep of
every submodule's git log found **five `develop → main` merge commits, all mine**
— see "The count was wrong" below. The two originally identified:

- `sellsuki-central-control-backend` **!321** — "Release: develop → main (CCS
  staging) — 56 commits, fast-forward". **56 commits, six authors, three epics.**
- `sellsuki-company-management-frontend` **!506** — "promote develop to main —
  BOLA Workspaces (BOLA-310/311/312) + MS-1345 Good Receive to staging".

`sellsuki-invitation` has none; rps's only one (!38) was closed and is pan.nit's.

**⚠️ The count was wrong — it is five, not two.** On 2026-09-17 a `/retro` swept
every submodule's git log for `Merge branch 'develop' into 'main'` and found five,
every one authored by me:

| repo | when |
|---|---|
| `backend/oc2plus-line-crm-service-3rdparty-api` | 2026-09-10 10:23 |
| `backend/oc2plus-line-crm-service-backoffice-api` | 2026-09-10 10:45 |
| `frontend/sellsuki-company-management-frontend` (CCS3) | 2026-09-10 12:01 |
| `backend/oc2plus-line-crm-service-backoffice-api` | 2026-09-11 04:23 |
| `frontend/oc2plus-linecrm-frontend-backoffice` | 2026-09-11 04:24 |

CCS **!321** does not appear in that list because it fast-forwarded (no merge
commit), so the real total is at least six events across five repos.

**Why this was missed the first time:** the first check looked only at the repos
named in the complaint. Three OC2Plus repos were doing the same thing and nobody
mentioned them, so they were never checked — the search was scoped to the
accusation instead of to the behaviour. Same shape as
[[feedback_verify_absence_claims]].

**And OC2Plus is dual-mainline too** — verified 2026-09-17 from the GitLab
deployments API, `main` → staging and `develop` → development in
`oc2plus-line-crm-service-backoffice-api`, `-member-api`, `-3rdparty-api`,
`oc2plus-linecrm-frontend-backoffice`, `oc2plus-linecrm-frontend-member` and
`oc2plus-line-crm-e2e-playwright` (staging only). So these four merges were not a
different, legitimate release flow; they were the same mistake in repos nobody
had checked. Note this sits alongside
[[feedback_oc2plus_merge_to_develop]]: day-to-day OC2Plus work targets `develop`,
which is exactly why promoting the whole of develop to main is so costly there.

**Status 2026-09-17: the fallout is CLOSED — do not re-raise it.** The owners of
the promoted work (kimzey and Pan on CCS3, Teerachai Pantip and Nithiphat
Kitsamret on the three OC2Plus repos, wayla on PAT-2634) know what happened and
have already gone through staging and repaired it. There is no outstanding
"go tell them" action. What stays live is the rule below, not the incident.

**Why it is wrong, in their terms:** in these repos `main` builds **staging** and
`develop` builds **dev**. A promotion MR moves everything sitting on develop to
staging at once — including other people's half-finished work — and nobody who
owns those changes agreed to release them.

Note !321's own description said *"I am not merging this… the call belongs to
whoever owns those epics"*. Flagging the risk did not help: opening the MR put
the merge one click away, and it was merged the same day. **The instinct to not
merge it should have been the instinct to not open it.**

**The same disease in a smaller dose:** opening a feature branch straight to
`main` also lands on staging without ever passing dev. I did that again on
2026-09-17 with CCS3 !519 (User List test ids) — merged to main, and it is
**not** on develop, so dev is behind and the next promotion can silently revert
it (see [[reference_fast_forward_merge_silently_reverts]] and
[[reference_stale_narrow_fix_mr_reverts_the_broad_one]]).

**The rule, confirmed by the user 2026-09-17:** *"เป็นการ merge จาก feature
branch เข้า develop กับ main แยกกัน"* — a change that belongs on both lines gets
**a feature branch merged into each line separately**. Never a `develop → main`
promotion MR: releasing everything sitting on develop is a decision for whoever
owns the work in it, not a side effect of one person wanting their own change on
staging.

Concretely, per change: branch off `origin/main` → MR to main, and branch off
`origin/develop` (or cherry-pick) → MR to develop, opened together. That is what
[[reference_rps_dual_mainline]] was already asking for — where develop-only left
staging without an internal API and cost a day of bisecting — just without
mistaking "both lines need it" for "promote the whole branch".

**⚠️ One branch, two MRs is NOT a shortcut for this — it fails as soon as the
lines diverge.** On 2026-09-18 PAT-2700 opened !130 (→main) and !131 (→develop)
from ONE branch cut off `origin/main`. !130 was fine; !131 came back
`cannot_be_merged`, conflict in `cmd/migration/migrations/migrations.go`.

Cause: the two lines already carried **twin commits** — the same change merged
separately into each line, with different SHAs (invite-release-usage !128/!129,
and wayla's payment.tool permissions). Both lines had independently added
migration `0023`, and develop additionally had `0024`. So a main-based branch
merged into develop drags main's twins along and collides on the migration
registry — with a real risk of dropping develop's `0024` during conflict
resolution.

Fix that worked: close the develop MR, cut a NEW branch off `origin/develop`,
`git cherry-pick` the two commits (clean, no conflict), open a fresh MR (!132).
Verify with `git diff origin/develop..HEAD -- cmd/migration/` returning **0
lines** and the registered migration count unchanged.

So the rule's "branch off each line separately" is literal. Reusing one branch
works only while the lines have not diverged, which is exactly the condition you
cannot rely on and will not notice until GitLab refuses the merge.

**Which repos this applies to** (verified 2026-09-17 from the deployments API,
`main` → staging and `develop` → development in every one): CCS3,
sellsuki-central-control-backend, role-and-permission, i18n-management-backend,
central-configuration-system, sellsuki-invitation, **and the whole OC2Plus
line-crm group** — `oc2plus-line-crm-service-backoffice-api`, `-member-api`,
`-3rdparty-api`, `oc2plus-linecrm-frontend-backoffice`,
`oc2plus-linecrm-frontend-member`. Treat this list as "every repo checked so
far", not "every repo": OC2Plus was absent from it for a week while five of these
merges sat in its history.

**Repos with ONE mainline — no twin needed:** sellsuki-chat-core,
sellsuki-ai-agent, ai-platform-kit-go (no `develop` branch at all).

**Two repos where `develop` is effectively dead — ask before twinning:**
`sellsuki-invitation` (develop last touched 2026-03-10, 0 commits ahead of main,
35 behind; dev env last deployed 2026-03-16) and
`central-configuration-system` (develop last touched 2026-08-14, diverged 54/85).
A twin MR there is not a cheap mirror, it is a merge into a stale branch.
