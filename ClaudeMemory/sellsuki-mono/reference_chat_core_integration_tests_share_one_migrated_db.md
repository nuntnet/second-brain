---
name: reference-chat-core-integration-tests-share-one-migrated-db
description: chat-core test/ integration tests that call migratedDB per test take ~90-200s each under parallel-session DB load; one migrated DB + fresh workspaces per subtest runs 12 ACs in ~3 min
metadata:
  node_type: memory
  type: reference
  originSessionId: 06c387cf-201d-45e5-b84b-e33e880023f0
  modified: 2026-11-04T10:43:28.817Z
---

In `backend/sellsuki-chat-core/test/`, `migratedDB(t)` (testdb.OpenIsolated + full migration chain) costs ~90 s per call when other sessions share the local Postgres, up to ~200 s. Twelve top-level tests timed out the 10-minute default.

Pattern that works (AI-380 `test/board_integration_test.go`, 2026-11-04): one top-level `TestX` builds the DB and server once, each AC is a `t.Run` subtest that creates fresh workspaces (`workspace_repository.Create`) and registers them in a shared `authctxtest.FakeWorkspaceResolver` map (maps are references, so the checker sees later entries).

Seed traps found the same day: `cases.assignee_user_id` is NOT NULL ("" = unassigned); AI-379's customer_list names people only when a member store is wired (nil members → every name `unresolved`).

Related: [[reference-parallel-sessions-share-test-postgres-spurious-route-test-failures]]
