---
name: reference-audit-action-is-closed-enum
description: sellsuki-go-logger/v2 AuditPayload.Action is a closed enum and the struct has no metadata field — Jira cards that name audit events cannot be implemented literally
metadata:
  type: reference
---

`log.AuditPayload` in `github.com/Sellsuki/sellsuki-go-logger/v2` constrains what an
audit entry can say:

- `Action` accepts only `AuditActionCreate|Update|Delete|Access` ("create"|"update"|"delete"|"access")
- there is **no free-form metadata field** — the payload is
  `ActorType, ActorID, Action, Entity, EntityRefs []string, EntityOwnerType, EntityOwnerID`

So a card that specifies event names (`RECEIPT_CLAIM_OCR_COMPLETED`,
`POINT_CLAIM_QUEUE_ACCESS_DENIED`) or a `metadata.{...}` object cannot be
implemented as written. The working pattern: put the outcome marker as the
**first `EntityRefs` entry** so it stays greppable, and flatten what the card
wanted in metadata into the remaining refs (`"mismatch=3"`, `"agreed=true"`).

Hit twice in OC2Plus (OC-4461, OC-4464) — the second time only because the
first was not written down. Note it in the commit and the Jira comment so the
PO sees the deviation rather than assuming the event name exists to query on.

⚠️ `EntityRefs` is a shared long-lived store: never put values read off a
customer document in it. See [[project_oc4464_ocr_vendor_decision]].
