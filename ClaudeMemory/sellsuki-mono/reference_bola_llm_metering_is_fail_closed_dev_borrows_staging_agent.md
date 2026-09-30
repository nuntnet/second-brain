---
name: reference_bola_llm_metering_is_fail_closed_dev_borrows_staging_agent
description: BOLA's llm_service (AI-227) returns "AI usage metering is not configured" on EVERY LLM call when AI_PLATFORM_KIT_BASE_URL/TOKEN are empty — chatbot cannot answer; there is no AI Agent on dev, so dev points at staging's (bola-dev MR !204)
metadata:
  type: reference
---

Verified 2026-09-30 in `src/repository/llm_service/llm_service.go` (`GenerateReply`, `outbox.ready()` gate).

- Fail-closed, not "quietly off": empty `AI_PLATFORM_KIT_BASE_URL`/`AI_PLATFORM_KIT_SERVICE_TOKEN` → every reply errors, both platform-funded (goes through the agent, `/internal/v1/llm`) and BYOK (calls the provider directly but reports usage to the agent first). I told the user and two MR descriptions the opposite ("metering ปิดเงียบ") and had to correct it.
- BOLA is a *client* of the shared AI Agent service (`backend/sellsuki-ai-agent`); it does not contain one.
- AI Agent is deployed **only on staging** (ns `sellsuki`, svc `sellsuki-ai-agent-svc:80`; the repo has `values-staging.yml` only). Nothing in `sellsuki-dev`.
- Decision 2026-09-30 (option B): dev borrows staging's agent — `bola-dev` MR !204 sets the two env vars like staging and the user copied Secret `bola-ai-platform-kit-secret` into `bola-dev` (no-echo pipe; value never read). Costs: dev metering rows land in staging's store; a staging credential is used by dev. Not proven that the staging agent accepts dev's company/workspace ids (possible 403) — check by making the chatbot answer once on dev. Replace with a dev agent (option C) when one exists.
- Reachability from a `bola-dev` pod to the staging agent works (no NetworkPolicy): `/system/readiness` 200, `/internal/v1/llm` 401 without token. Health path is `/system/readiness`, not `/health`.
- Missing Secret key = `CreateContainerConfigError` — create the Secret before merging anything that references it.
