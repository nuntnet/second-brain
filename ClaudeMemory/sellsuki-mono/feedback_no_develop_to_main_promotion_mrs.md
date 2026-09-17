---
name: feedback_no_develop_to_main_promotion_mrs
description: Other devs called out opening develop→main release MRs as wrong — it pushes unvalidated work onto staging in one shot; verified I opened two of them (CCS !321, CCS3 !506) on 2026-09-10
metadata:
  type: feedback
---

**Reported 2026-09-17:** other dev teams told the user his workflow was wrong —
*"ไม่ควรจะ MR จาก develop to main เลย เพราะจะทำให้มีของที่ยังไม่เรียบร้อยไปขึ้นบน staging"*
— naming central-control-backend, sellsuki-invitation, CCS3 and
role-and-permission.

**Checked, and it is about my MRs.** Two `develop → main` MRs exist and both are
mine, both opened and merged 2026-09-10:

- `sellsuki-central-control-backend` **!321** — "Release: develop → main (CCS
  staging) — 56 commits, fast-forward". **56 commits, six authors, three epics.**
- `sellsuki-company-management-frontend` **!506** — "promote develop to main —
  BOLA Workspaces (BOLA-310/311/312) + MS-1345 Good Receive to staging".

`sellsuki-invitation` has none; rps's only one (!38) was closed and is pan.nit's.

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

**How to apply:** default to targeting `develop`. Do not open a develop→main
promotion MR — releasing to staging is a decision for whoever owns the work in
it. When a repo genuinely needs a change on both lines (see
[[reference_rps_dual_mainline]], where develop-only left staging without an
internal API and caused a 503 bisect), open a feature branch onto **each** line
rather than promoting the whole of develop.
