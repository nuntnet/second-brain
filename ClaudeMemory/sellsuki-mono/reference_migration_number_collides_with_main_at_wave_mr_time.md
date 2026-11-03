---
name: reference_migration_number_collides_with_main_at_wave_mr_time
description: "Before opening a wave/long-lived branch MR, merge-tree it against current origin/main — another team can take the same rps/CCS migration number the same day; CCS MIGRATE_ONLY picks the FIRST prefix match"
metadata:
  node_type: memory
  type: reference
  originSessionId: f746364c-7c1d-4189-93bc-f067e13be78d
  modified: 2026-11-03T08:51:55.538Z
---

2026-11-03: the AI chat MVP wave branch carried rps `0037_seed_chat_bundle_permissions` and CCS
`_0030_migrate_permission_chat_bundles…`. OC-4770 merged rps `0037_…` and CCS `_0030_…` into `main`
that same morning. Ledger ids differed (both would run), but CCS `migrate_roles_permissions`
`MIGRATE_ONLY` matches **by prefix and takes the first hit**, so the runbook's `MIGRATE_ONLY=0030`
would have run the other team's backfill. CCS's own `TestMigrateOnly0023SelectsTheRagMigration`
checks prefix uniqueness and goes red after the merge — so CI would have caught it, an hour later.

**How to apply:** right before opening an MR for a branch that has sat a while (wave branches,
develop-side merge branches), run per repo:
`git merge-tree --write-tree --name-only origin/<wave> origin/main` and
`git diff --name-only $(git merge-base …) origin/main | grep -i migrat`.
A conflict in `migrations.go` means a number race. Renumber the side not yet applied anywhere but
local (that was ours), push the rename into the develop-side branch FIRST (so develop does not get the
other team's main-only work), then merge origin/main into the wave branch. Also grep for the old
number in comments of consumers (kit `authctx/bundle.go` G1 gate, chat-core) — they named `_0030`.
Related: [[reference_inbox_py_ids_collide_across_branches]].
