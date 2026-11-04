---
name: reference-gorm-raw-scan-embedded-model-fields-empty
description: "gorm Raw(\"SELECT c.*, joined cols\").Scan into a struct embedding the model leaves the embedded fields (even id) empty — no error"
metadata:
  node_type: memory
  type: reference
  originSessionId: d718c366-d986-4bd7-807c-ab83ab2106bb
  modified: 2026-11-04T10:45:54.121Z
---

In chat-core (AI-365, 2026-11-04), `Raw("SELECT c.*, s.id AS session_id, ... FROM cases c LEFT JOIN LATERAL ...").Scan(&[]struct{ postgresGormCaseModel; SessionID *string ... })` returned rows whose embedded model fields were all empty (id, unsorted_reason, ...) while the extra columns were filled. No error; tests only caught it by asserting on the row's id and reason.

**How to apply:** for a list read that needs model rows plus joined extras, scan the extras with `c.id AS row_id` into a flat struct, then load the models with `Where("workspace_id = ? AND id IN ?").Find(&models)` and join in Go. Scanning `c.*` straight into `[]Model` (no embedding) works.

Related: in chat-core `test/`, `migratedDB(t)` per test costs minutes on a loaded machine — run a card's AC tests as subtests of one `TestXxx` sharing one migrated schema, and assert on the test's own rows (shared-schema counts include other tests).
