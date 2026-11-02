---
name: reference_go_test_skips_underscore_dirs_ccs_migration_tests
description: "go test ./... skips directories starting with _ — CCS cmd/migrate_roles_permissions/migrations/_00NN and migrate_preset_roles/_00NN tests never run in a ./... run (likely not in CI either); run them by explicit path"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2ed39e79-b1a0-4aac-9faa-c61a3445c73a
  modified: 2026-11-02T11:46:30.964Z
---

Found 2026-11-02 (AI-359): CCS migration packages are named `_0023…`, `_0030…`, `_0004…`. The go tool ignores
directories beginning with `_` or `.` in `./...` patterns, so their tests are silent in a full run.

**How to apply:** run them by path, e.g.
`go test ./cmd/migrate_roles_permissions/migrations/_0030_migrate_permission_chat_bundles_to_preset_roles_and_manage_holders/`,
and say in the ledger that you did. Locally CCS also needs `GOPROXY=https://proxy.golang.org` (a module zip is
missing from the cache: charmbracelet/x/ansi@v0.10.1).
