---
name: monorepo-no-origin
description: "sellsuki_mono root git repo is local-only — its sole remote \"glab-base\" points at the BOLA repo, so never push monorepo branches"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-08T07:38:57.028Z
---

**Corrected 2026-08-08:** the root repo now HAS a real remote — `origin` → `git@gitlab.sellsuki.com:sellsuki/sellsuki/monorepo.git` (verified live with `git ls-remote`). Monorepo branches CAN be pushed to `origin`.

The trap that remains: `glab-base` → `sellsuki/bola/back-office-of-line-api-backend.git` is also configured, and pushing there would publish workspace content into the unrelated BOLA backend repo. **Always push the monorepo explicitly to `origin`, never to `glab-base`, and never with a bare `git push` if the branch's upstream is unset.**

Consequences:
- Monorepo-level work (submodule ref updates, `docs/*`, scaffolds not yet promoted to their own GitLab project) exists **only in the local clone**. Committing is the only durability step available; there is nothing to push to.
- Every `backend/*` and `frontend/*` submodule DOES have a proper GitLab remote — push there as normal.
- Example: the AI-97 ETL scaffold lives on local monorepo branch `feature/AI-97-etl-scaffold` (`backend/ai-analytics-etl/`) awaiting someone with GitLab permission to create `sellsuki/data-pipeline/ai_chat_data_pipeline` — see [[project-ai-sprint234-autonomous-run]].
