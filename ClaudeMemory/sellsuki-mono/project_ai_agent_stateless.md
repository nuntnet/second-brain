---
name: project_ai_agent_stateless
description: sellsuki-ai-agent has NO database — chat-core owns workspace config + the eval harness; config travels on the reply request
metadata: 
  node_type: memory
  type: project
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-12T16:55:24.038Z
---

Decided 2026-08-12. `backend/sellsuki-ai-agent` is **stateless apart from Redis**
(circuit breaker only). It has no Postgres, no migrations, no backfill.

- **Workspace AI config** (model, persona, temperature, confidence threshold,
  negative keywords, guardrail/disclosure/degrade templates, allowlist domains,
  kill switch) lives ONLY in chat-core's `workspaces` table. chat-core reads it
  and sends it in `AIAgentRequest.Config` on every reply call.
- **Eval harness** (golden set, runs, case results, override audit, judge) lives
  ONLY in chat-core — `src/use_case/eval` + migrations 0060-0063 +
  `workspace_repository.UpdateAIConfigWithEvalOverride`. Kept there because the
  gate and the config write are ONE transaction; an HTTP gate would orphan runs.
- ai-agent's copies of both were duplicates left behind by the E3 extraction
  (it copied, didn't move) and are deleted.

**Why this keeps mattering:** the split-brain produced an admin console writing
to a row the reply path never read, and nothing failed. If a future card says
"ai-agent reads/stores workspace config" or "add a migration to ai-agent", that
card predates this decision.

Fail-closed direction moved with the config, in two places:
- chat-core can't read the workspace → does not call the agent at all
- ai-agent gets a request with empty `Model` → degrades (Model, not SystemPrompt:
  an empty persona is a legitimate state)

Related: [[project_ai_mvp_integration]], [[project_ai_chat_platform_plan]]
