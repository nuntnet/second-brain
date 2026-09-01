---
name: project_ai115_conversation_goal_backend_only
description: AI-115 conversation goal (frequency cap + conversion link) is fully built in chat-core + ai-agent but has zero frontend, so no admin can reach it
metadata:
  type: project
---

AI-115 "[E3][Goals] Workspace-defined conversation goal — frequency cap + conversion
event" is **merged and complete on the backend**, and **completely absent from the
frontend**. Jira says "In Review", which as usual means merged-unverified.

Backend on `origin/main` of `backend/sellsuki-chat-core`:
- `GET /v1/workspaces/{id}/conversation-goal` (Operator scope)
- `PUT /v1/workspaces/{id}/conversation-goal` (Company Admin, optimistic lock via `version`)
- routes live inside `route/workspace/` — **not** in a `route/conversation_goal/`
  directory, which is why a directory-shaped search finds nothing
- `src/entity/conversation_goal/`, `src/repository/conversation_goal_repository/`,
  `src/use_case/conversation_goal{,_runtime}.go`, migration `0087_add_conversation_goals`
- `backend/sellsuki-ai-agent` returns structured goal decisions (commit e253b5a)

Config shape: `key`, `enabled`, `link`, `max_prompts_per_conversation` (the frequency cap).

Frontend `frontend/ai-chat-admin-frontend`: **0 commits mentioning AI-115, and
`git grep ConversationGoal origin/main` returns nothing** — no port, no adapter,
no page, no i18n key.

This is NOT the same feature as the Playbook. Two distinct "goal" concepts exist:
- **AI-115 conversation goal** = one conversion target + how often the assistant
  may push it (has API, no UI)
- **AI-127 Playbook / Checkpoint** = the multi-step conversation pipeline, computed
  from facts, never stored — see [[project_ai_conversation_intelligence]]. Has both
  API and UI at Settings → Brain → Playbook.

Neither is a numeric KPI target; searching `target_leads`/`monthly_target`/`kpi`/
`conversion_target`/`revenue_target` across the whole stack returns nothing.

**Why:** an admin cannot configure the conversation goal at all today, and nothing
in the UI hints it exists — the textbook case of
[[reference_backend_without_frontend_is_invisible]].

**How to apply:** when asked what workspace setup is missing, AI-115's UI is a real
gap with the backend already waiting. Also do not describe the Playbook as "the goal
feature" without saying which of the two you mean.
