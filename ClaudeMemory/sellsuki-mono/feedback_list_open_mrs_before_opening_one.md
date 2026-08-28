---
name: feedback-list-open-mrs-before-opening-one
description: "List a repo's open MRs immediately before opening one — listing at session start is not enough when parallel Claude sessions are working the same board"
metadata:
  node_type: memory
  type: feedback
---

Before opening an MR in a repo, list that repo's **currently** open MRs. Listing once at the start of a session is not enough.

**Why:** 2026-08-25 I diagnosed the `ApplyPlaybookVersion` 23505 bug from CI job 235532 and opened chat-core !23 at ~15:00. A parallel Claude session had already opened **!22 for the same bug at 13:18** — and I had exchanged scope with that session earlier the same day. !23 merged first, which left !22 conflicting through no fault of its author, and cost a third MR (!24) to consolidate.

Worth recording alongside the mistake: **both diagnoses were identical without contact** — same root cause (unlocked `MAX(version)` read at READ COMMITTED, primary key as the real arbiter) and both deterministic tests independently seeded version 2 with `is_current = false`. Two blind reads landing on the same mechanism is good evidence the mechanism is right; it is not a reason to keep the implementation that merely landed first. Theirs was better (target-less `clause.OnConflict{DoNothing:true}` + `RowsAffected`, no new deps) so !24 adopted it and dropped mine.

**How to apply:** re-list right before `POST /merge_requests`. When a duplicate is found after the fact, compare implementations on merit and consolidate onto the better one, saying plainly on both MRs what happened and why. See [[reference_parallel_sessions_duplicate_symbols]] for the code-level version of this collision.

## Listing the MRs is not enough — an MR title hides the cards inside it

2026-08-28, `sellsuki-messaging-backend`. I listed the open MRs, saw !14 titled
`AI-176: establish messaging/FB integration baseline`, concluded it was
unrelated, and opened !25 for the AI-120 OpenAPI gate. !14 already contained
the gate — `51e9097 ci(contract): fail the pipeline on a breaking OpenAPI
change (AI-120)` — plus three follow-up fixes for the exact problems I then
re-derived from scratch.

**A branch routinely carries cards its MR title never names.** Before starting
work on a card, search the branch commits, not the MR titles:

```bash
git log --oneline --all --grep="AI-120"
git log --oneline origin/<mainline>..origin/<each-open-mr-branch> | grep -i <key>
```

Also worth checking `git log --all --grep="<feature phrase>"` — commits here
frequently omit the Jira key entirely (chat-core's whole answer-trace feature
landed under "design 2a", so `--grep=AI-154` found nothing).

**When the duplicate is mine, the newer one is not automatically worse.** !14's
gate had three commits fixing what I hit later: run it with `bash` (so
`read -d ''` is safe), `--unshallow` on a shallow checkout, and a `found` vs
`compared` split so a branch adding the repo's first specs is not failed. My
version lost on two of those three. Compare on merit and say so plainly —
including retracting criticism written before reading the other side's CI job
definition, which is what made me wrongly claim !14 had the dash bug.

Related: [[reference_silent_semantic_merge_break]], [[reference_ai_board_in_review_means_merged_unverified]].
