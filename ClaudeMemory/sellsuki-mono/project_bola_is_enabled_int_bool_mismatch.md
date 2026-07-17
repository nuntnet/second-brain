---
name: project_bola_is_enabled_int_bool_mismatch
description: "BOLA recurring bug — is_enabled columns declared INTEGER in raw-SQL migrations but bool in GORM model → pgx 500 on Postgres, masked by SQLite tests"
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
---

**Bug class in bola-backend (confirmed 2026-07-17):** several tables declare `is_enabled` (and similar boolean flags) as `INTEGER` in raw-SQL migrations, but the GORM model fields them as Go `bool`. On **Postgres** this 500s on INSERT/UPDATE:
`failed to encode args[N]: unable to encode true into binary format for int4 (OID 23): cannot find encode plan` (pgx binary protocol has no bool→int4 encoder).

**Why it hides:** unit/integration tests run on **SQLite** (glebarez/sqlite) which is loosely typed — bool into an INTEGER column just works — so tests pass green while the feature is broken on real Postgres. First real Postgres CREATE reveals it.

**Status per table:**
- `auto_push_messages.is_enabled` — INTEGER (migration 0011) vs model bool → **BROKEN**, fixed MR !126 (migration `0125_auto_push_is_enabled_boolean`). This blocked auto-push-to-group create (see [[project_bola_workspace_scoping_bugs]] sibling BOLA-270 QA card).
- `auto_replies.is_enabled` — INTEGER (0002) vs model bool → **BROKEN** (same MR !126). NOTE: migration 0005 re-declares the table via GORM `AutoMigrate` with a bool-tagged model, but **AutoMigrate never ALTERs an existing column's type** (only adds missing columns) — so the column stayed `integer`.
- `ai_chatbot_configs.is_enabled` — INTEGER (0018) but model is `int` with explicit boolToInt/intToBool at the entity boundary → **NOT a bug**.
- `webhook_settings.is_enabled` — flagged same shape BUT webhook settings create works fine on staging → likely already boolean / false positive (needs empirical repro before touching).

**Fix pattern:** Postgres-only, idempotent migration guarded on `information_schema.columns.data_type='integer'`, no-op on SQLite dialect:
```
ALTER TABLE <t> ALTER COLUMN is_enabled DROP DEFAULT;
ALTER TABLE <t> ALTER COLUMN is_enabled TYPE boolean USING (is_enabled <> 0);
ALTER TABLE <t> ALTER COLUMN is_enabled SET DEFAULT true;
```

**Lesson for new bool fields:** either model as `int` + convert at boundary (like ai_chatbot), OR migrate the column to real `boolean`. Never trust SQLite green tests for Postgres type correctness — verify bool columns empirically against Postgres. Related: [[reference_bola_jira_project]].
