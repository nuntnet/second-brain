---
name: pg-partial-index-onconflict-generic-plan
description: Postgres ON CONFLICT with explicit arbiter on a PARTIAL unique index fails with 42P10 from the 6th execution per connection (generic plan) — use target-less DO NOTHING
metadata: 
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-06T04:57:23.627Z
---

Postgres gotcha (bit us in chat-core AI-32 `GetOrCreateOpen`, fixed on branch `fix/AI-32-getorcreateopen-generic-plan`):

`INSERT ... ON CONFLICT (cols) WHERE <predicate> DO NOTHING` targeting a **partial unique index** works for the first 5 executions of the prepared statement on a connection, then fails every time with `SQLSTATE 42P10` — `plan_cache_mode=auto` switches to a generic plan after 5 executions and the arbiter index can no longer be inferred. With connection pooling this looks like a service that degrades under load and "heals" when connections recycle.

**Fix:** use a target-less `ON CONFLICT DO NOTHING` (no Columns/TargetWhere in GORM's `clause.OnConflict`) — needs no arbiter inference, applies to any unique index incl. partial ones — then re-read the row for get-or-create convergence. Alternatives: `SET plan_cache_mode=force_custom_plan` or disabling prepared-statement cache (both wider blast radius).

**Regression-test pattern:** run the statement 10+ times sequentially on a single connection (`sqlDB.SetMaxOpenConns(1)`); tests that run it ≤5 times will never catch this.

Applies to any repo using GORM + partial unique index upserts (multi-tenant "single open row per key" patterns like [[project_oc2plus_primary_invariant_pattern]]).
