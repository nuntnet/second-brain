---
name: entity-lib-tenant-kinds
description: shared backend/entity lib owns the IsActor allowlist that gates every rps tenant kind — how to add one, and the local-submodule-is-stale trap
metadata:
  node_type: memory
  type: reference
  modified: 2026-08-23T15:10:00.000Z
---

`gitlab.sellsuki.com/sellsuki/sellsuki/backend/entity` (GitLab project **536**) owns `IsActor()` / `IsResource()`. **rps validates BOTH `Actor.Kind` and `Tenant.Kind` against `IsActor()` before consulting Keto** — a kind missing there makes `CheckPermission`/`AssignRole` return `InvalidArgument`, which callers often swallow into a silent `false`.

**Why:** any service wanting a tenant tier below company (workspace/store/branch) is gated by one string in this lib — it looks like an architecture problem but is a one-line shared-lib change.

**How to apply:**
- Convention `<product>.<thing>`; existing tenant tiers under company: `patona.store`, **`bola.workspace`** + `bola.instance` (added 2026-08-01 via entity **!39**, BOLA-118 — the canonical template: constant + IsActor + `entity_test.go` + `access_control/namespace_test.go` + `identity/identity_test.go` + README).
- **Every team opens the MR themselves** with the patch; there is no Jira board for the rps/platform team. Branch naming: `feat/<TICKET>-<slug>`.
- `ValidateKind` has **no charset rule** (only `IsActor`); `ValidateId` requires `^[a-z\d_\-.]+$`. Kind names are frozen after tag — no rename path.
- ⚠️ **Local submodules lie about the version.** `backend/sellsuki-role-and-permission-management-backend` locally pins entity `v0.22.0` while its **main** is on `v0.28.0`; `ai-platform-kit-go` pins `v0.22.0`. Always read `go.mod` from GitLab main (`glab api projects/652/repository/files/go%2Emod/raw?ref=main`) before concluding a kind is unavailable.
- Turning a tier on takes TWO steps: merge+tag entity, then rps bumps the dep and deploys. Step 2 has no ticket anywhere — chase the !43 reviewer.
- AI chat platform's ask: `sellsuki.chat_workspace` — entity **!43** opened 2026-08-23 (AI-182). Would be the only kind with an underscore; flagged in the MR for the repo owners to confirm before merge.

Related: [[ai-backlog-gap-sweep-202608]], [[bola-rbac-keto-direct]], [[rps-is-system-role-trap]]
