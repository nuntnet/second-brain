---
name: gorm-pgx-libpq-automigrate
description: "GORM AutoMigrate fails with \"got 2 parameters but the statement requires 1\" when a lib/pq pool is handed to the pgx-based postgres driver — and only on the SECOND boot"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-08T10:16:39.687Z
---

`gorm.io/driver/postgres` is built on **pgx**. If you construct it with a `*sql.DB` pool opened by **lib/pq**, `AutoMigrate` blows up with:

```
pq: got 2 parameters but the statement requires 1
```

**The trap is the timing:** GORM only reaches the disagreeing code path when the table **already exists**, so the very first boot succeeds and silently arms the failure for every boot afterwards. Found in `sellsuki-messaging-backend` (Aug 2026) — the service had been unrestartable since its chat tables were first created, and nobody noticed because nobody restarted it. The crash is ordered, so only the first unfixed table surfaces at a time; fixing one reveals the next.

**Fix:** give the GORM postgres driver its own pgx-based connection (or open the pool with the pgx stdlib driver), don't reuse a lib/pq pool.

**Regression test that actually catches it:** call `AutoMigrate` **three times** against a real Postgres in one test — a single call on a fresh DB passes even with the bug present.

Related in-repo gotcha class: [[pg-partial-index-onconflict-generic-plan]] (also only fails after repeated execution).
