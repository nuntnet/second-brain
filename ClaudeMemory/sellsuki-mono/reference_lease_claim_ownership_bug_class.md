---
name: lease-claim-ownership-bug-class
description: "Recurring bug class — worker claim/lease code releases or completes by row id without verifying it still owns the lease, causing double-processing"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-07T18:26:58.325Z
---

Recurring across this org's Go and Python workers (hit 3× in Aug 2026 sprints, by different authors, in different repos). Shape:

1. A worker `claim`s rows with a lease (`claimed_at`, `status='processing'`), and a reaper re-claims rows whose lease expired.
2. The completion path (`release` / `MarkDone` / `MarkFailed`) updates **by row id only** — no `AND claim_token = ?` / owner check.
3. A slow worker that overruns its lease then finishes **clears the new owner's claim**, causing double-processing (double export, duplicate ingest, duplicate sends) and silently resetting state.

Companion bug in the same code: the second phase of a two-phase `SELECT ... FOR UPDATE SKIP LOCKED` claim omits re-checking `status='idle'/'pending'`, so a competitor that committed between phase 1 and phase 2 gets the row claimed twice.

**Fix pattern:** issue the completion as a conditional update — `WHERE id = ? AND claim_token = ?` (or lease generation) — and treat 0 rows affected as "lost the lease", handled without writing. Re-check the status predicate inside the locked phase.

**Why tests miss it:** every occurrence needed two concurrent workers plus an interleaved write; single-worker tests and fakes that mirror the unguarded semantics both pass. Regression tests must use concurrent goroutines/tasks with a forced lease expiry.

Occurrences: chat-core `sheet_export_repository` (AI-65), rag-core `sheet_connector_repository.release/claim_due` (AI-44), and the AI-39 `dequeue_batch` lineage both inherited from. Related: [[pg-partial-index-onconflict-generic-plan]] is the other DB-level gotcha that only shows up after repeated execution.
