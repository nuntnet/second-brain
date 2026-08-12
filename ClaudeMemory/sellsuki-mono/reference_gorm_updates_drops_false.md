---
name: reference_gorm_updates_drops_false
description: "GORM Updates(&struct) silently skips zero values — a bool going false never persists, the UPDATE succeeds, and every layer reports success (hit on BOLA workspace suspend 2026-08-13)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-12T17:32:39.472Z
---

`db.Where(...).Updates(&model)` with a **struct** only writes non-zero fields.
So setting a bool to `false`, a string to `""`, or a number to `0` writes
**nothing** — and the statement still succeeds, `RowsAffected` can be 1, and the
use case, the route and the client all report success.

Found in `bola-backend` `workspace_repository.UpdateWorkspace`: suspending a
workspace set `is_active=false`, which never reached the database. QA reported
"suspend does nothing" and the persistence half was invisible from every log.
(A `gorm:"default:true"` tag on the column makes it worse — the same omission on
INSERT is what the default is *for*, so the tag reads as intentional.)

Fixes, in order of preference:
1. `Select("*").Omit("id", "created_at").Updates(&model)` — correct when the use
   case does load-then-mutate and hands back the whole entity (it did here).
   Widening what the statement writes puts more weight on the WHERE clause, so
   test that a second row is untouched.
2. A `map[string]any` of just the changing columns.
3. Pointer fields on the model.

**How to test it:** a mock repository cannot catch this — the bug is in the SQL
GORM decides to emit. Use an in-memory SQLite round trip (`glebarez/sqlite`,
already a dep in bola-backend) and read the row back. Verify the test is red
without the fix; it is the only signal that exists.

Related: [[project_bola_is_enabled_int_bool_mismatch]], [[reference_flag_without_enforcement]]
