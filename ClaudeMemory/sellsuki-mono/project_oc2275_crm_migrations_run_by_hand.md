---
name: project-oc2275-crm-migrations-run-by-hand
description: "the OC2Plus CRM schema repo has zero CI pipelines — merging a migration does nothing, someone must run db-migrate by hand per environment"
metadata:
  node_type: memory
  type: project
---

**This is the OC2Plus norm, not a defect in one repo.** Checked 2026-09-09: all
six have a `.gitlab-ci.yml`? No — and zero pipelines each:
`oc2plus-line-crm-migration`, `oc2plus-line-crm-migration-kafka`,
`migration-oc2plus-service-{core-api,chat,file,messaging}`. Running by hand is
the convention. (The README claims "Run migrations as part of CI/CD pipelines",
which describes a state nobody built — corrected in MR !81/!82 along with a
RUNBOOK.)

`sellsuki/oc2plus/line-crm/migration/oc2plus-line-crm-migration` (db-migrate,
Node) owns the CRM Postgres schema — **not** the service repos, and not
`line-crm/backend/entity`. Asking GitLab for its pipelines returns `[]`: there is
no CI at all, so **merging a migration changes nothing**. Someone runs
`npm run migrate:stg:up` by hand, per environment.

**Correction (2026-09-09):** the staging half of the story below was wrong —
that migration WAS applied to staging on 1 Sept 07:00. The "never ran" reading
came from querying a stale Postgres pod; see
[[reference-datastore-stale-postgres-pod]]. What stands: this repo (and every
OC2Plus migration repo) has no CI, so merging still changes nothing and someone
runs it by hand.

Cost of not knowing this (2026-09-01): migration `20260809000000` adding
`api_key.name` / `expires_at` / `created_by` and widening `key` to varchar(64)
was merged to both `main` (!74) and `develop` (!73) in August and never run. The
API-key management screens returned **500** on staging for weeks
(`pq: column i.name does not exist`), and 3rdparty-api's own key verification
selects `expires_at`, so partner auth was broken the same way.

**How to apply:**
- A card that needs a schema change is not done when the migration merges.
- `db-migrate up` runs **every** pending migration, not just yours — always
  `-- --dry-run` first; nobody knows how far behind an environment is.
- Postgres is in-cluster (`postgresql.datastore:5432`), so a `kubectl
  port-forward` reaches it; credentials are in secret `oc2plus-crm-secret` (ns
  `octoplus` / `octoplus-dev`). `database.json` has no `port` field, which is
  why a forward on a non-default port needs one added.
