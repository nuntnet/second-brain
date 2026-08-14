---
name: reference_chatcore_migrations_break_at_0066
description: "chat-core migrations die at 0066 on any clean DB — local dev sits at 0065, so AI-127's checkpoint tables (0067) exist NOWHERE; also the workspace_repository Postgres tests are contention-flaky"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 369f9f55-f1f1-4ed5-92cb-b4d1e841aa8f
  modified: 2026-08-14T14:46:22.673Z
---

Found 2026-08-14 while checking whether renumbered migrations applied (AI-157).

**`go run ./cmd/migration` on an empty database stops at `0066_case_type_sales_lead`**
with `ERROR: op ANY/ALL (array) requires array on right side (SQLSTATE 42809)`.
Reproduced on the integration line AND on pristine `a781bfb` — it is not caused by
any in-flight branch.

Cause: `src/repository/case_repository/postgres_gorm_model.go`
`MigrateRenameSeededCaseType` writes `array_replace(allowed_types, ?, ?)` and
`WHERE ? = ANY(allowed_types)`, but `case_type_configs.allowed_types` is
**`text`** (verified via `information_schema` on both a live and a fresh DB), and
0064 seeds the bare string `sales_lead`. The statement fails regardless of data —
the model and the migration disagree about the column type.

**Consequences that bite silently:**
- The local dev `chat_core` DB has migration records only through
  `0065_add_case_id_to_leads`. **0066 and 0067 are absent.** So AI-127's
  Playbook/Checkpoint tables have never existed in any database — do not assume
  that feature is exercisable locally just because its card looks delivered.
- Anything queued after 0066 is unreachable, including Tier 1's renumbered
  0068-0071 (see [[project_ai_merge_topology_risk]] and AI-141).
- Any NEW environment — fresh staging deploy, new dev machine, CI with a clean DB
  — cannot migrate. It survived because nobody migrates from zero.

Do not "fix" it by guessing the encoding: whether `allowed_types` is meant to be
`text[]`, a single value, or a JSON array is AI-126's call, and guessing wrong
corrupts workspace case-type config.

**Unrelated but same session:** the `src/repository/workspace_repository`
Postgres integration tests are **contention-flaky** against the shared local
Postgres (which also serves the running stack and other sessions' worktrees).
They failed on one heavily-parallel `go test ./src/...` run (different tests each
time — `Rename`, `UpdateAIConfig`, `UpdateURLAllowlistConfig`) and passed on
three other runs including `-p 1`. A red there is not automatically your change;
re-run the package alone before believing it.
