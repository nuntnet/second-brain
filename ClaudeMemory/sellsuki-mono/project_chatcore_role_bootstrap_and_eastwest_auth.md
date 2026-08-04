---
name: chatcore-role-bootstrap-and-eastwest-auth
description: "chat-core's CHAT_WORKSPACE_OPERATOR_ROLE_ID replaced with self-bootstrap-by-name (matches management-backend's pattern); SERVICE_TOKEN confirmed as the right east-west auth choice"
metadata: 
  node_type: memory
  type: project
  originSessionId: a862be5f-18d0-42e0-a03d-7e69c1c914b9
  modified: 2026-08-04T02:54:56.746Z
---

**Role provisioning pattern (2026-08-04):** No other service in the monorepo hardcodes a role id from `sellsuki-role-and-permission-management-backend` into deployment config — grepped every `values-*.yml`/`.env.example`, only `sellsuki-chat-core`'s `CHAT_WORKSPACE_OPERATOR_ROLE_ID` did this, and it had never actually been populated anywhere (even local dev left it unset).

The established, real pattern (verified in `backend/management-backend/src/use_case/company.go:203-237`, an already-deployed service talking to the exact same gRPC backend) is **lookup-by-name + self-bootstrap at runtime**: call `ListRoles`, scan for the role by `Name`, and if missing call `CreateRole` right then and cache the returned id — no env var, no manual per-environment setup step, no secret. Fixed chat-core's `role_assignment_repository` to do the same (branch `feat/ai-17-role-self-bootstrap`) — removed `CHAT_WORKSPACE_OPERATOR_ROLE_ID` from `cmd/chat_core_server`, `.env.example`, and all 3 `deployment/values-*.yml`.

**Why this matters beyond chat-core:** any NEW AI chat platform service (E1-E9) that needs a preset role from `sellsuki-role-and-permission-management-backend` should follow this same lookup-by-name-then-create pattern, not invent a `*_ROLE_ID` env var — it was chat-core's own one-off mistake, not an established convention worth repeating.

**East-west auth for the new platform (SERVICE_TOKEN, confirmed 2026-08-04):** compared chat-core's `SERVICE_TOKEN` (via `ai-platform-kit-go/authctx.ServiceTokenMiddleware` — shared secret header, constant-time compared) against the real production RBAC service's own `/internal/*` routes (`sellsuki-role-and-permission-management-backend/src/interface/fiber_server/route/v1/route_internal.go`): that service has **zero app-layer auth** on its internal routes today, relying purely on network-level isolation (not exposed via the public gateway, cluster-internal-only traffic only) — its own code comment flags this as a known gap, explicitly *not yet hardened* because retrofitting a shared token requires coordinating every existing consumer (CCS3, bola-backend) at once, a hard rollout problem.

**Conclusion: keep `SERVICE_TOKEN`.** It's not redundant or over-engineered — it's actually ahead of what the org's own most-similar existing service does, and since chat-core is new (no existing consumers to break), it gets defense-in-depth from day one instead of inheriting the RBAC service's retrofit problem. `ai-platform-kit-go/authctx.ServiceTokenMiddleware` is the sanctioned shared mechanism for east-west auth across this whole new AI platform — reuse it, don't invent a per-service alternative.

**LLM_API_KEY storage (2026-08-04, user-confirmed):** intentionally a single platform-wide secret (Vault/K8s Secret), never per-workspace-configurable — per AI-14's own design ("W1 must have one LLM client interface to prevent quota bypass before E5/AI Gateway exists"). If a UI is ever wanted to manage/rotate this key, it must be a **System Admin-only screen on CCS1** (`sellsuki-central-control-backend`), never a workspace-admin-facing config field — user's own call, matches the security reasoning.

Related: [[project_ai_chat_platform_plan]]
