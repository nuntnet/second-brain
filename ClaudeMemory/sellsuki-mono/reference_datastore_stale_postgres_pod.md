---
name: reference-datastore-stale-postgres-pod
description: "ns datastore holds more than one Postgres — the live one is whatever svc/postgresql points at (pg18), and postgresql-postgresql-0 serves a stale copy of the same database under the same name"
metadata:
  node_type: memory
  type: reference
---

`POSTGRES_HOST` for every OC2Plus service is the **Service**
`postgresql.datastore`. On staging that Service selects
`app.kubernetes.io/instance=postgresql-pg18` → pod **`postgresql-pg18-0`**.

The namespace also runs **`postgresql-postgresql-0`**, an older instance that
still holds databases with the *same names* (`staging_oc2plus_crm`,
`development_oc2plus_crm`) frozen at an older state.

Cost of picking the pod by name (2026-09-09): the stale pod reported `api_key`
with 8 columns, `key` as varchar(32), 0 rows, and a migration ledger stopping
31 July — so I reported to the user, who forwarded it to QA, that staging was
blocked on an unrun migration and no API key could be minted. The live instance
had that migration applied since **1 Sept 07:00**, `key` at varchar(64), and
four API keys already created. `db-migrate --dry-run` had said "No migrations to
run" and I doubted the tool instead of the pod.

**How to apply:** never `kubectl exec` into a Postgres pod chosen by name.
Resolve it from the Service first:

```bash
kubectl -n datastore get endpoints postgresql \
  -o jsonpath='{range .subsets[*].addresses[*]}{.targetRef.name}{"\n"}{end}'
```

or better, port-forward `svc/postgresql` and connect as the application user —
that is the account whose view of the schema the running service actually gets.
Documented in the migration repo's RUNBOOK.md (MR !81/!82). See also
[[project-oc2275-crm-migrations-run-by-hand]], whose "staging never ran it"
conclusion this corrects.
