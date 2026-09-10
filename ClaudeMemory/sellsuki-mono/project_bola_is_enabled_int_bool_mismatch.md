---
name: project_bola_is_enabled_int_bool_mismatch
description: BOLA has a recurring bug class — migration declares a flag column INTEGER while the GORM model binds Go bool, so every Postgres write 500s while reads look fine; swept and closed 2026-09-10
metadata:
  type: project
---

bola-backend migrations repeatedly declare boolean flag columns as
`INTEGER NOT NULL DEFAULT 1/0` while the repository model binds them as a Go
`bool`. `gorm.io/driver/postgres` uses pgx's extended protocol, so the parameter
goes out as OID 16 and Postgres refuses:

```
ERROR: column "is_active" is of type integer but expression is of type boolean
```

**Writes fail, reads do not** — `database/sql` converts an `int64` of 0/1 into a
`bool`. So list/get endpoints look healthy and only create/update 500. That
asymmetry is why instances survive for months.

**The SQLite test suite cannot see it.** SQLite has no rigid column type, so all
2880 tests in `./src/...` pass identically on broken and fixed code. There is no
Postgres service in `.gitlab-ci.yml`, so no CI job exercises the real type. A
test passing is not evidence about this class — see
[[reference_test_stub_more_permissive_than_service]].

**How to sweep it:** compare every `bool` field in `src/repository/**` (extract
`TableName()` + the `gorm:"column:X"` tag) against `information_schema.columns`
on a migrated Postgres. 21 bool fields exist; 20 resolve to a real column
(the 21st is a scan-only struct).

**History**
- migration `0125_auto_push_is_enabled_boolean` fixed `auto_push_messages` and
  `auto_replies.is_enabled`.
- migration `0130_bool_columns_integer_to_boolean` (MR !174, 2026-09-10) fixed
  the remaining four: `registration_forms.is_active`,
  `rich_menu_assignments.is_default`, `lon_subscribers.is_friend`,
  `feedback.notion_pushed`. This is what made `POST /v1/registration-forms`
  return 500 on production from migration `0033` onward.
- As of that MR the class is closed for the code that existed then, but nothing
  prevents the next one. Tables created after ~0123 correctly use `BOOLEAN`.

Related: [[reference_gorm_updates_drops_false]] is a different GORM/bool trap
(zero-value skipping in `Updates`), not this one.
