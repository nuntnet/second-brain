---
name: monorepo-no-origin
description: "sellsuki_mono root git repo is local-only — its sole remote \"glab-base\" points at the BOLA repo, so never push monorepo branches"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-07T05:29:14.329Z
---

The `~/sellsuki_mono` root repository has **no correct remote**. `git remote -v` shows only `glab-base` → `sellsuki/bola/back-office-of-line-api-backend.git`, which is a leftover pointing at an unrelated repo. Pushing a monorepo branch there would publish workspace content into the BOLA backend repo — never do it.

Consequences:
- Monorepo-level work (submodule ref updates, `docs/*`, scaffolds not yet promoted to their own GitLab project) exists **only in the local clone**. Committing is the only durability step available; there is nothing to push to.
- Every `backend/*` and `frontend/*` submodule DOES have a proper GitLab remote — push there as normal.
- Example: the AI-97 ETL scaffold lives on local monorepo branch `feature/AI-97-etl-scaffold` (`backend/ai-analytics-etl/`) awaiting someone with GitLab permission to create `sellsuki/data-pipeline/ai_chat_data_pipeline` — see [[project-ai-sprint234-autonomous-run]].
