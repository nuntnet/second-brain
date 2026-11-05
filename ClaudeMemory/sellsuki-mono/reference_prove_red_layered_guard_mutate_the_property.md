---
name: prove-red-layered-guard-mutate-the-property
description: Prove-red on a layered guard (chat-core ClaimBatch) — removing one layer stays green; break the property itself, and give a stub latency so concurrency is real
metadata:
  type: reference
---
On AI-399 (2026-11-05) the "never sent twice" test stayed GREEN when SKIP LOCKED + the SELECT lease predicate were dropped from chat-core `ClaimBatch`: its UPDATE re-asserts the predicate (M8 defense in depth) and refuses the batch. That proves the second layer, not the test.

**How to apply:** for an idempotency/concurrency test, mutate the property the test is about (e.g. `DefaultClaimLeaseDuration = -time.Minute`), not one of several guards. And give the httptest stub a delay (100ms) so "overlapping cycles" actually overlap — without it the cycles serialize and a broken lease can still look fine. Related: [[feedback_prove_red_script_never_pipe_into_head]].
