---
name: project_oc4267_standalone_no_qms
description: OC-4267 workspace binding descoped to standalone (no QMS/quota/deactivate) 2026-07-02; QMS reconnects later as follow-up; enum now pending/active/failed
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

**OC-4267 (BOLA workspace binding) was descoped 2026-07-02** — confirmed by dev Patjukom (Jira comment 42530), card rewritten + replied (comment 42533):

- **QMS/quota/plan cut entirely**: binding is standalone — trigger = internal API / admin action (NOT "QMS plan assigned"), calls CCS `POST /v1/companies/:id/bola-workspaces` directly with `{name, slug}` (no planId), no quota debit/credit, no quota-exhausted reject. OC-4258 changed from prerequisite → follow-up.
- **Company deactivate/delete cut**: OC2Plus CRM has no company deletion → revoke/deactivate flow deferred.
- **binding_status canonical NOW = `pending / active / failed`** — `failed` = terminal when `binding_attempts >= MAX_RETRY` (stops retry; manual retry via OC-4291 UI). `revoked` and `quota_exhausted` will only exist in the QMS follow-up.
- **Why:** ship binding first; QMS quota (OC-4258) reconnects later as a separate follow-up card — the card has a "🔮 QMS integration — follow-up" section listing exactly what returns (plan-assigned trigger, debit/credit, quota_exhausted terminal, revoked).

⚠️ **Code drift:** the earlier OC-4267 implementation in backoffice-api (commit `4c8a2e4`) DOES include QMS debit + deactivate — dev must strip/gate those to match the descoped card. Card is source of truth. Affects [[project_oc_bola_domain_boundary]] (OC-4291 badge enum) — OC-4291's `revoked` badge row is now follow-up-only.

**Execution decisions (2026-07-06, answered dev teerachai.pan's 4 questions, comments 42628→42629):**
1. **Call path = BOLA direct interim** (`POST /v1/workspaces` with `X-System-Token`) — because CCS's `POST /v1/companies/:id/bola-workspaces` + `AllocateBolaWorkspace()` EXISTS but only on unmerged branch **`feature/provider`** (verified: `git branch -r --contains` shows only that branch; deployed CCS lacks it). Must be built as an **adapter/repo with config base URL** so it can switch BOLA→CCS without touching the use case.
2. ⚠️ **Migration debt:** workspaces created direct have NO record in CCS `bola_workspaces` table, which provider UI (OC-4263) + invite-admin flows read → the CCS-migrate follow-up MUST include a **backfill**.
3. `revoked` enum: keep in code/DB unused (no drop migration) — returns in QMS follow-up.
4. `SeedBolaQuota` (RegisterBolaQuota + loop AssignBolaQuota) removed from startup entirely; startup keeps only `RetryPendingBindings` (5-min).
5. Trigger = `POST /v1/user/company` (CreateCompany) → `BindBolaWorkspace` async directly — every company gets a workspace, no quota gate.
6. Scope stays in OC-4267 (no new card); follow-ups = QMS reconnect (with OC-4258) + CCS migrate/backfill.
