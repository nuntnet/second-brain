---
name: reference_testify_mock_panic_in_fiber_handler_kills_test_binary
description: "A testify mock's 'Unexpected Method Call' panic inside a fiber app.Test handler crashes the whole go test binary — prove-red output shows neither --- FAIL nor ok; run the targeted test alone"
metadata:
  type: reference
---

Seen 2026-10-29 on AI-213 (chat-core route tests over `repository.MockCheckpointRepository`).

A mutation that made the code call the mock with an argument no `.On(...)` matched (e.g. the wrong workspace id) panics
inside the fasthttp goroutine that `app.Test` runs. Go's test runner cannot attribute it, so the binary dies in the FIRST test
that hits the mock, and a `grep "^--- FAIL|^ok"` over the run prints **nothing** — which reads as neither proof nor failure.
The isolation test built to catch that mutation never ran.

**How to apply:** when a prove-red mutation prints no `--- FAIL` and no `ok`, look for `panic: mock: Unexpected Method Call`
and rerun with `-run <the test meant to catch it>` alone; only a named `--- FAIL` from that run counts
(same rule as `[setup failed]` in feature-dod). Related: [[reference_testify_permissive_default_wins]].
