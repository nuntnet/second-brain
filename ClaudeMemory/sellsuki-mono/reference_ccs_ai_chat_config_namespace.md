---
name: ccs-ai-chat-config-namespace
description: "CCS already hosts the AI platform's config (AI-19) with versioning/rollback/audit/Keto/cache-invalidation — but only company + workspace scope, no provider scope"
metadata:
  type: reference
---

`backend/central-configuration-system` already has an **AI Chat Platform namespace built by AI-19**, and it covers most of what a new AI-platform config card would otherwise build from scratch.

Files: `src/interface/grpc_server/configuration_service/config_service_grpc_ai_chat.go`, `src/use_case/ai_chat_config.go`, `src/use_case/model/ai_chat_config.go`

**Scopes that exist (`model/ai_chat_config.go:19-20`):**
- `ai-chat-company`  → `Service=ServiceAIChatCompany`, `UserId=""`, `Location=companyID`
- `ai-chat-workspace` → `Service=ServiceAIChatWorkspace`, `UserId=workspaceID`, `Location=companyID`
- **No provider-level scope.** That is the gap for anything platform-wide (e.g. which models are enabled, default model per tier).

**What CCS already gives you — do not rebuild these:**
- versioning + `GetListVersionConfig` + `RollbackConfig` RPCs
- audit with real before/after: `src/entity/config_audit` has `ActorID`, `Before`, `After`, `Version`, `Action` (create/update/rollback), `Reason`, `CreatedAt`
- server-side permission checks on every AI-chat RPC (the generic `Internal*` RPCs skip them; the ai_chat surface does not)
- optimistic locking — `ErrConfigVersionConflict` maps to `codes.Aborted` so a caller can retry
- **active Redis cache invalidation on write**, so an edit is visible on the next resolve, not "eventually within TTL"
- `ResolveAIChatWorkspaceConfig` — the single API other services call; workspace overrides win over company, then schema defaults
- backing store is etcd, cache is Redis

`ai_chat_config.go` describes itself as "deliberately additive", so adding a scope should not disturb other namespaces.

**How to apply:** before proposing a new config table in any AI-platform service, check whether it belongs here instead. Relevant to AI-134, whose card assumes a table in `sellsuki-ai-agent` — a service whose README states it deliberately has no database. Credential *values* still belong in Vault (see [[ai-88 credential work]]); Vault KV v2 also supplies version + created_time natively, so credential metadata needs no table either.
