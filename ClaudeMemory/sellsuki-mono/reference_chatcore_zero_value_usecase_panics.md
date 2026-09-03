---
name: chatcore-zero-value-usecase-panics
description: chat-core route tests using &use_case.UseCase{} SIGSEGV for any use case that opens a span — tracer is set only inside New()
metadata:
  type: reference
---

Verified 2026-09-03 in `backend/sellsuki-chat-core`.

`UseCase.tracer` is assigned **only** inside `use_case.New(...)`. So
`&use_case.UseCase{}` has a nil tracer, and any method whose first line is
`uc.tracer.Start(...)` panics with a nil-pointer dereference.

**Why this is confusing rather than obvious:** several route tests in
`src/interface/fiber_server/route/*` use `&use_case.UseCase{}` and are
perfectly fine — `kb_entry`, `case_type_config`, and every `openapi_test.go`
— because those use cases never touch the tracer. Copy that pattern into a
package whose use case DOES (`fact_schema`, `ads_spend`, `anti_abuse`,
`assignment`, `accessible_*`, and most others) and you get a SIGSEGV thrown
from inside the fiber handler, several frames deep, with the stack pointing at
`use_case/<name>.go` rather than at anything the test wrote.

**Fix:** build it with `use_case.New(nil, nil, nil, nil, nil, nil, nil, nil,
nil)` and then the setter for whatever the route needs. `otel.Tracer` returns a
no-op when no provider is registered, so `New` costs nothing in a test.

Not a production bug: `cmd/chat_core_server/main.go` is the only construction
site and it uses `New`. Which is exactly why nobody had hit it — and why it
will keep catching people writing the FIRST test for a route package.

Check before copying a harness: `grep 'uc.tracer' src/use_case/<file>.go`.

Related: [[chatcore-is-the-admin-bff]], [[chatcore-route-tests-timeout-under-load]]
