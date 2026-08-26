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
