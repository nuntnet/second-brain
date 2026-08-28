---
name: pg-partial-index-onconflict-generic-plan
description: ON CONFLICT with an explicit arbiter on a PARTIAL unique index fails 42P10 — sometimes only from the 6th execution, sometimes on the first; use target-less DO NOTHING
metadata: 
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-08T03:16:05.065Z
---

Postgres gotcha (bit us in chat-core AI-32 `GetOrCreateOpen`, fixed on branch `fix/AI-32-getorcreateopen-generic-plan`):

`INSERT ... ON CONFLICT (cols) WHERE <predicate> DO NOTHING` targeting a **partial unique index** works for the first 5 executions of the prepared statement on a connection, then fails every time with `SQLSTATE 42P10` — `plan_cache_mode=auto` switches to a generic plan after 5 executions and the arbiter index can no longer be inferred. With connection pooling this looks like a service that degrades under load and "heals" when connections recycle.

**Fix:** use a target-less `ON CONFLICT DO NOTHING` (no Columns/TargetWhere in GORM's `clause.OnConflict`) — needs no arbiter inference, applies to any unique index incl. partial ones — then re-read the row for get-or-create convergence. Alternatives: `SET plan_cache_mode=force_custom_plan` or disabling prepared-statement cache (both wider blast radius).

**Regression-test pattern:** run the statement 10+ times sequentially on a single connection (`sqlDB.SetMaxOpenConns(1)`); tests that run it ≤5 times will never catch this.

**Sweep for siblings — this shipped 3× in chat-core alone** (`GetOrCreateOpen` twice over, then `workspace_repository` `EnsureDefault` on the live auto-provision path). Each time it was fixed in isolation without grepping. When you fix one, run `grep -rn "TargetWhere" src/repository/` and check every `clause.OnConflict{` with a non-empty `Columns` against the model's index definitions. A CI guard test enforcing this was added on `fix/s6-trackA-review-fixes`.

Applies to any repo using GORM + partial unique index upserts (multi-tenant "single open row per key" patterns like [[project_oc2plus_primary_invariant_pattern]]).

---

**2026-08-28 — correction: it does not always wait for the generic plan.**

Measured on `point_claim_ocr_result` (OC-4464), whose unique index is
`(company_id, point_claim_id) WHERE deleted_at IS NULL`:

```sql
INSERT ... ON CONFLICT (company_id, point_claim_id) DO NOTHING
-- ERROR: there is no unique or exclusion constraint matching the ON CONFLICT
--        specification          ← on the FIRST execution, via psql
```

So "safe because it worked in dev" is not evidence either way — depending on the
index and how the statement is sent, it can fail immediately OR survive five
executions and then start failing. Treat any explicit arbiter against a partial
index as broken, full stop.

Target-less `ON CONFLICT DO NOTHING` matches a partial index fine and is the
form to use.

Note for Go: **goqu makes the mistake hard** — `goqu.DoNothing()` has no
`Target` method, so it cannot be expressed through the query builder at all.
Raw SQL still can. That also means a mutation test "flip to an explicit target"
will simply fail to compile rather than go red, so it proves nothing — verify
with raw SQL against the real schema instead.

