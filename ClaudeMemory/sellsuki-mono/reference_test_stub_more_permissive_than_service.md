---
name: reference-test-stub-more-permissive-than-service
description: "An httptest stub that answers any method on any path lets a wrong URL/verb pass its test and fail in production — make the stub reject what the real service rejects (Kratos recovery 405 case)"
metadata:
  node_type: memory
  type: reference
---

Found 2026-08-14 in `bola-backend` BOLA-314, already merged to `main`:
`send-password-reset` returned **500** in the browser —
`kratos recovery: submit returned status 405: Method Not Allowed`.

Cause: only Kratos flow **creation** has an `/api` variant.
`GET /self-service/recovery/api` starts an API flow; the **submit** goes to
`/self-service/recovery`. Verified against the local Kratos:

```
GET  /self-service/recovery/api    → 200   create flow
POST /self-service/recovery/api    → 405   what the code did
POST /self-service/recovery?flow=… → 200   correct (flow response's ui.action says so)
```

**The part worth remembering is why a test did not catch it.** The test *asserted
the wrong path and passed*:

```go
assert.Equal(t, []string{
    "GET /self-service/recovery/api",
    "POST /self-service/recovery/api",   // green, and wrong
}, paths)
```

Its `httptest.NewServer` handler answered **any method on any path**. A stub more
permissive than the service it stands in for does not test the contract — it only
records what the code did. Fix was to make the stub return **405 for POST on
…/recovery/api**, exactly as Kratos does; the old URL then fails.

**How to apply:** when stubbing an external HTTP service, encode its *refusals*
(wrong verb, wrong path, unknown id), not just its happy path. Same family as
[[reference_testify_permissive_default_wins]] — a permissive stub is a test that
passes on broken code. And prefer probing the real local service to confirm a URL
shape before writing the stub around your assumption.
