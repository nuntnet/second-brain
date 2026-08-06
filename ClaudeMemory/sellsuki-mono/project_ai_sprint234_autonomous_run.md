---
name: project-ai-sprint234-autonomous-run
description: Sprint 2-4 AI-chatsystem cards ALL implemented on pushed feature branches WITHOUT MRs (run finished 2026-08-06) — full branch map per repo; cards left In Progress pending review/MR
metadata: 
  node_type: memory
  type: project
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-06T04:57:55.034Z
---

User directive (2026-08-05): implement ALL Sprint 2/3/4 cards of project AI, committing continuously to feature branches, **no MRs opened**. Run COMPLETED 2026-08-06 — every card below is implemented, tested green, pushed, and left **In Progress** in Jira (final comment on each card has AC coverage + deferred items). Future sessions: unmerged ≠ unstarted — check these branches before re-implementing. Merge order matters (stacks must merge bottom-up).

- `backend/sellsuki-messaging-backend` (NEW submodule 2026-08-05) — 7-deep unmerged stack: main ← `feature/AI-23-normalized-message-schema` (MR!10) ← `feature/AI-29-chat-module-isolation` (MR!11) ← `feature/AI-24-fb-app-registry` ← `feature/AI-25-webhook-ingress` ← `feature/AI-26-page-token-lifecycle` ← `feature/AI-27-fb-send-api` ← `feature/AI-28-fb-media`. Gates: make unit-test-otp + make guard-test. Vault/Kafka/Redis clients are hand-rolled minimal (no shared org client exists).
- `backend/sellsuki-chat-core` — stack: main ← `feature/AI-33-server-exposure` (WS+REST+Redis fan-out, seq cursor) ← `feature/AI-34-tier0-router` ← `feature/AI-47-lead-aggregate` (lead bounded context, guard narrowed) ← `feature/AI-56-confidence-escalation` ← `feature/AI-20-observability` (hop metrics + /metrics + SRE handoff doc). Off main separately: `feature/AI-18-vault-gateway` (secrets contract guard, verify_secret_keys, replicaCount fix, docs/vault-gateway-handoff.md) and `fix/AI-32-getorcreateopen-generic-plan` (see [[pg-partial-index-onconflict-generic-plan]]; AI-33's test workaround capping loops at 4 should be removed when it merges). Migrations 0004-0007 live in the stack.
- `backend/central-configuration-system`: `feature/AI-19-chat-config-namespace` (ai-chat-company/-workspace schemas, resolve+rollback+audit; found latent bug: generic InternalRollBackConfiguration deletes history instead of creating a new version).
- `backend/ai-platform-kit-go`: `feature/AI-16-customer-fact-schema`.
- `backend/rag-core` (NEW submodule 2026-08-05, GitLab sellsuki-rag/poc/rag-core): `feature/AI-38-workspace-namespace` (new rag_knowledge Milvus collection + knowledge_chunk mirror — Milvus kept, NOT pgvector, per spike) ← `feature/AI-39-ingest-pipeline` (chunk→embed→upsert worker, Postgres-backed queue). Needs live-Milvus verification.
- `frontend/sellsuki-invitation`: `feature/AI-21-invitation-i18n-ux` — based on top of open MR!16/!17 (i18n infra existed from a parallel session, NOT reflected in Jira comments at the time). CCS invites are LINK-based, no email field.
- AI-36 RAG spike deliverable: `docs/ai-rag-inventory-recommendation.md` (monorepo docs; decided Milvus stays, workspace_id is the only tenancy gap, ingest greenfield).
- SRE handoffs (not code): AI-18 Vault paths/Emissary mapping (docs/vault-gateway-handoff.md), AI-20 dashboards/alerts (docs/observability-handoff.md), AI-33 WS ingress timeout config. See [[project-ai-platform-deploy-gating]].
