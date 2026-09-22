---
name: reference_consent_service_owner_id_is_a_bare_uuid
description: "sellsuki-service-consent writes owner_id as the BARE company uuid, not the platform's sellsuki.company:<uuid> token — and GET /consent returns {results,total} with a 10-item default page and no tenancy of its own"
metadata:
  type: reference
---

Measured against the live staging service on 2026-09-22, not read from a spec.

```
GET /consent/4084
{"id":4084,"name":"Register","type":"pdpa","status":"published",
 "owner_id":"f2bf928c-7cd0-4059-9326-82fddeba4933","active":true,"app_id":3,"version":1}
```

🔴 **`owner_id` is the bare company uuid.** It is NOT
`sellsuki.company:<uuid>` — the token this platform uses everywhere else and
that `product_repository` builds for PIS. The consent service's own spec
documents a composed `sellsuki.provider:<uuid>-sellsuki.company:<uuid>`, which
the live data does not use either. Three spellings in play; the service writes
the simplest one.

Cost of assuming otherwise: backoffice-api's document picker filtered with
`strings.Contains(owner_id, "sellsuki.company:"+companyID)` → false for every
document of every company → **the picker was empty everywhere**, and the screen
showed its own "no documents yet" empty state, which is the exact thing the
feature exists to remove. Fixed in backoffice-api MR !621 by matching the uuid
alone (the composed form contains it, so both spellings work).

**This was the SECOND time the same card shipped an empty picker for the same
reason.** The first was a status allow-list written as `on`/`enabled`/`active`
when the service writes `published`. Both times the code carried a fluent,
plausible comment explaining the choice; neither had been compared with a real
row. See `.claude/rules/verify-at-the-boundary.md` §1 and §5 — this is the
canonical example for that rule.

**Other shapes worth knowing about `GET /consent`:**
- response is `{"results":[...],"total":N}` — not `data`, and not a bare array
  (backoffice-api's decoder accepts `consents`, `results`, and a bare array).
- **no params → first 10 of 2097 on staging.** It applies no tenancy at all, so
  every caller gets other companies' documents and must filter locally.
- backoffice-api calls it with **no query string at all** — no `owner_id`, no
  `limit`. So a company whose documents are not in the newest 10 still sees an
  empty picker even after !621. Not yet carded as of 2026-09-22.
- `type` is NOT a filter this endpoint has; passing it is silently ignored.
  Accepted params (its own v1.yaml): owner_id, app_id, offset, limit, status,
  active, name.

Statuses seen live: `published` (bindable). The service's `entity/consent.Status`
has five: on, off, draft, published, unpublished.

See [[reference_oc2plus_consent_binding_has_no_writer]] ·
[[project_oc4089_consent_binding_page]]
