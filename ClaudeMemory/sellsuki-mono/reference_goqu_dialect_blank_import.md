---
name: reference_goqu_dialect_blank_import
description: goqu.Dialect("postgres") without the blank dialect import silently emits ? placeholders — compiles, unit-tests pass, fails only on a real DB
metadata:
  type: reference
---

**`goqu.Dialect("postgres")` returns the DEFAULT dialect unless the package is
blank-imported:**

```go
_ "github.com/doug-martin/goqu/v9/dialect/postgres"
```

Without it, `.Prepared(true).ToSQL()` emits `?` placeholders instead of `$1`, and
pq rejects the statement with a misleading **`pq: syntax error at or near ")"`**.

**Why it is nasty:** it compiles, `go vet` is clean, and every use-case test
passes — because those mock the repository. The only symptom appears the first
time a query reaches a real database. Hit on 2026-09-12 writing
`member_tier_repository` in `oc2plus-line-crm-service-backoffice-api`; found only
by running `cmd/tier_sweep` against the local `oc2plus_crm`.

**How to apply:** when adding a new `*_repository/postgresql.go` in any goqu repo
here, copy the import block from an existing one (e.g.
`api_key_repository/postgresql.go:12`), not just the struct. Cheap guard that
needs no DB — assert on the generated SQL:

```go
sql, _, _ := goqu.Dialect("postgres").From("t").Select("c").
    Where(goqu.C("x").Eq("v")).Prepared(true).ToSQL()
// must contain "$1" and must NOT contain "?"
```

See [[reference_oc2plus_backoffice_codegen_is_go]] · [[project_oc3559_member_tier_state]]
