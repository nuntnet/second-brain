---
name: reference_ambiguous_404_fail_open
description: "Mapping every 404 to \"resource not found\" makes a cross-service security check fail open during a deploy window — decide on error_code, not status"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-12T15:29:53.573Z
---

A client that treats **any** 404 as "the resource does not exist" cannot tell
these apart:

```
service says no such record   → {"error_code":"INVITATION_NOT_FOUND"}   (404)
service has no such endpoint  → "Cannot GET /internal/v1/…"             (404)
```

Hit for real in CCS (BOLA-311, 2026-08-12): the accept path calls
`GET /internal/v1/invitations/{code}` to check an invitation was addressed to the
person accepting it, and treated 404 as "nothing to enforce". For the whole
deploy window where CCS ships ahead of rps, the endpoint is missing → 404 → the
security check silently passed **every** invitation. The commit message claimed
it failed closed; it failed open.

Rules that follow:
- In a client, branch on the **error_code / body**, not the status, whenever the
  status alone is ambiguous. A bare 404 must surface as an error naming the
  deploy order.
- Any check whose failure mode is "skip the check" needs a test that a MISSING
  endpoint does not look like a negative answer. Fiber's default 404 body is
  `Cannot GET <path>` with no JSON and no `error_code`.
- resty only unmarshals into `SetResult`/`SetError` when the response carries
  `Content-Type: application/json` — a test server using `json.NewEncoder(w)`
  without setting that header parses to an empty struct and quietly takes the
  wrong branch.

Found by auditing my own work, not by a failing test — the class is invisible to
tests that only exercise the deployed-in-order case.

Related: [[reference_lease_claim_ownership_bug_class]], [[project_ai_platform_deploy_gating]]
