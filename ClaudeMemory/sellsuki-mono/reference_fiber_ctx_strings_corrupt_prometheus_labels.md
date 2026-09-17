---
name: reference_fiber_ctx_strings_corrupt_prometheus_labels
description: ctx.Method()/ctx.Path() alias a per-connection buffer — storing them as Prometheus label values yields garbage labels, duplicate children, and a /metrics that 500s entirely, hiding every metric in the service
metadata:
  type: reference
---

Fiber's `ctx.Method()`, `ctx.Path()`, `ctx.Get(...)` return strings that **alias
fastttp's per-connection buffer**. They are valid only for the duration of the
handler. Anything that outlives the handler must be copied:

```go
method := strings.Clone(ctx.Method())
path   := strings.Clone(strings.Replace(ctx.Path(), "/", "", 1))
```

A Prometheus label value outlives the handler — the collector keeps it forever.
Without the clone, the buffer is reused by the next request on that connection
and the stored label mutates into garbage (`method="GETT"`, truncated or
concatenated paths).

## Why the blast radius is the whole endpoint

Corrupted labels create **duplicate children** for the same metric. `Gather()`
then returns an error, and promhttp answers the request with **500 for the
entire `/metrics` endpoint** — not just the broken family. Every unrelated
metric in the service disappears at once.

So the symptom is not "one metric looks odd". It is **"`/metrics` is 500 and I
have no observability at all"**, which reads like a scrape/infra problem and
sends you looking in the wrong place. Fixed in messaging-backend MR !57
(`src/interface/fiber_server/helper/metrics.go`); the same shape is possible in
every Fiber service in this workspace.

## The test that actually proves it

Not a single request. Fire concurrent requests with **differing method and path
lengths** over reused connections (240 requests × 5 methods × 4 path lengths),
then assert `Gather()` succeeds **and** every observed label is one that was
actually sent. Verified pass 3/3 with the fix, fail 3/3 without — a
single-request test stays green either way.
