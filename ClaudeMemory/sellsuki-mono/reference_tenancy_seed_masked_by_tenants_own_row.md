---
name: tenancy-seed-masked-by-tenants-own-row
description: "A two-workspace tenancy test stays green with a join predicate dropped when tenant A has its own matching row (or A's row comes first on disk); seed so only B's row can match"
metadata:
  node_type: memory
  type: reference
  originSessionId: 6d416b86-7ae5-47ba-9ab1-ff959fe23b7c
  modified: 2026-11-05T08:54:22.479Z
---

Found on AI-380's follow-up store (2026-11-05). The test seeded B's lead and follow-up rows naming A's case id. It still passed with these predicates dropped:
- `l.workspace_id = ?` on the leads join. A also had its own lead, so the join produced A's row plus B's terminal row, and the terminal filter quietly removed B's.
- `workspace_id = ?` in the row-lock read (`LIMIT 1`). A's row was inserted first, so the read returned A's row anyway.

**How to apply:** in a tenancy seed, give A no row of its own on the table under test, so that only B's row could ever match. Insert B's rows BEFORE A's, so that any first-row or `LIMIT 1` read finds B first. Then prove each predicate red **separately**: one predicate dropped per run. Related: [[verify-at-the-boundary]].
