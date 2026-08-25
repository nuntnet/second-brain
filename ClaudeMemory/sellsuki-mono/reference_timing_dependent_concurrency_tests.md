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
