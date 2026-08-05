---
name: project-ai-sprint234-autonomous-run
description: Sprint 2-4 AI-chatsystem cards implemented on pushed feature branches WITHOUT MRs (user directive 2026-08-05) — branch map per repo + messaging-backend stacked on unmerged AI-29 branch
metadata: 
  node_type: memory
  type: project
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-05T16:08:17.190Z
---

User directive (2026-08-05): implement ALL Sprint 2/3/4 cards of project AI, committing continuously to feature branches, **no MRs opened**. Future sessions must not assume unmerged = unstarted — check these branches before re-implementing:

- `backend/sellsuki-messaging-backend` (NEW submodule added 2026-08-05): chat-module work is a **stack of unmerged branches**: main ← `feature/AI-23-normalized-message-schema` (MR!10) ← `feature/AI-29-chat-module-isolation` (MR!11) ← `feature/AI-24-fb-app-registry` (+ AI-25/26/27/28 planned on top). FB cards must base on the AI-29 branch, not main.
- `backend/sellsuki-chat-core`: `feature/AI-33-server-exposure` (WS+REST+Redis fan-out); AI-34/47/56 + AI-18 config-check queued after. OpenAPI route-parity gate on main means every new route needs openapi.yaml updates.
- `backend/central-configuration-system`: `feature/AI-19-chat-config-namespace`.
- `backend/ai-platform-kit-go`: `feature/AI-16-customer-fact-schema`.
- `frontend/sellsuki-invitation`: `feature/AI-21-invitation-i18n-ux` (i18n infra from scratch + invite mgmt UX; CCS invites are LINK-based, no email field — AC8 premise mismatch documented on card).
- AI-36 RAG spike deliverable: `docs/ai-rag-inventory-recommendation.md` (monorepo docs, decides AI-38/39 approach).
- Deferred/SRE-blocked: AI-18 (Vault/gateway — mostly SRE), AI-20 (observability), see [[project-ai-platform-deploy-gating]].
