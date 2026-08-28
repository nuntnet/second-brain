---
name: timing-dependent-concurrency-tests
description: A goroutine-race test that passes locally against unfixed code proves nothing — stage the end state the lost race produces instead
metadata: 
  node_type: memory
  type: reference
  originSessionId: 270e02aa-6ae1-461a-ada0-6fb19a8e9e87
  modified: 2026-08-25T13:19:04.096Z
---

Concurrency tests written as "spawn N goroutines and assert exactly one wins"
are **unfalsifiable on a fast local Postgres** — the racers serialise on their
own and the test goes green against code that is genuinely broken. It then only
fails on CI (slower ARM runners), where it reads as a flake and gets re-run
instead of fixed.

Hit on 2026-08-25 with chat-core `TestApplyPlaybookVersion_ConcurrentApply_ExactlyOneWins`:
green locally 3/3 against the unfixed code, red on CI job 235525.

**The technique that works:** don't race — *stage the end state the lost race
produces*, then take the same code path deliberately. For a check-then-insert,
that means pre-inserting the row that the winner would have written, arranged so
the optimistic-lock check still passes, so the caller reaches the insert with its
key already taken. Deterministic, and it reproduces the CI error verbatim.

**The bug shape underneath** (recurring here — see [[lease-claim-ownership-bug-class]]):
a plain `SELECT` for the current version takes no lock, so two writers both read
N and both aim at N+1. A demote `UPDATE` in between does **not** serialise them:
under READ COMMITTED the loser re-evaluates its `WHERE` after the winner commits,
matches nothing, locks nothing, and proceeds. The unique index is the only real
serialisation point — so let it decide and translate the violation into the
domain error the contract already promises.

Use the repo idiom — target-less `clause.OnConflict{DoNothing: true}` then
`RowsAffected == 0` (`lead_repository:89`, `workspace_repository:112`). Naming a
conflict target reintroduces [[pg-partial-index-onconflict-generic-plan]].

Always verify red→green explicitly: stash the fix, confirm the new test fails
with the *same* error the CI log shows, restore, confirm green.

## Sibling trap: zero-latency fakes hide cache-freshness bugs (2026-08-28)

Same family, opposite direction. A test for "the write's response must land in
the cache synchronously" (`setQueryData` vs `invalidateQueries`) **passes with
the fix removed** when the fake port resolves instantly: the invalidate's
refetch settles in the same microtask as the assertion, so the cache reaches the
new value either way. Written that way it asserts nothing.

Fix: make the read *stall* for the window under test — a fake with a
`freezeReads()` that parks every read after the initial load, so the only thing
that can refresh the cache is the write's own response. Then mutation-test it:
remove the production line, confirm RED, restore, confirm GREEN. Do that check
every time — the first draft of the AI-199 guard was green both ways.

Rule of thumb: if a test is about *when* a value arrives, a fake with no latency
cannot express it.
