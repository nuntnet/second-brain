---
name: project-ai-sprint234-autonomous-run
description: Sprint 1-5 AI-chatsystem cards ALL implemented + code-reviewed + review-fixed on pushed branches WITHOUT MRs (run finished 2026-08-07) — branch map, merge order, review-fix branches
metadata: 
  node_type: memory
  type: project
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-07T03:51:09.782Z
---

User directive (2026-08-05, extended 2026-08-06): implement ALL Sprint 2/3/4 then Sprint 5 cards of project AI, plus adversarial code review of everything + fix the findings — committing continuously to feature branches, **no MRs opened**. Run COMPLETED 2026-08-07. Every card is implemented, tested green, pushed, reviewed, and its review findings fixed. Cards sit **In Review** in Jira (each has AC-coverage + review + fix comments). Future sessions: unmerged ≠ unstarted — check these branches before re-implementing.

**Merge order matters** — stacks merge bottom-up, and see [[project-ai-chatcore-merge-order]] for the chat-core CRITICAL prerequisite.

- `backend/sellsuki-messaging-backend` — stack: main ← `feature/AI-23-normalized-message-schema` (MR!10) ← `feature/AI-29-chat-module-isolation` (MR!11) ← `feature/AI-24-fb-app-registry` ← `feature/AI-25-webhook-ingress` ← `feature/AI-26-page-token-lifecycle` ← `feature/AI-27-fb-send-api` ← `feature/AI-28-fb-media` ← **`fix/fb-stack-review-fixes`** (3 CRITICALs fixed: cross-tenant app_secret overwrite, unauthenticated registry API, media-fetcher SSRF; + 9 MAJORs). Gates: `make unit-test-otp`, `make guard-test`.
- `backend/sellsuki-chat-core` — stack: main ← AI-33 ← AI-34 ← AI-47 ← AI-56 ← AI-20 ← AI-35 ← AI-54 ← AI-55 ← AI-58 ← **`fix/chat-core-review-fixes`** (cross-tenant lead_data JOIN, unrecoverable escalation, prompt-layer forgery, tier0 language no-op, fail-open takeover guard, unbounded queries, metric cardinality, WS re-backfill). Separately off main: `feature/AI-18-vault-gateway`, `fix/AI-32-getorcreateopen-generic-plan`. Migrations 0004–0012.
- `backend/central-configuration-system`: `feature/AI-19-chat-config-namespace` (+ review fixes: PATCH semantics via new DeepMergeConfig helper, codes.Aborted on version conflict; generic CCS rollback deletes history = latent gap for other namespaces).
- `backend/ai-platform-kit-go`: `feature/AI-16-customer-fact-schema` (+ fixes: sensitive/scope flip ratchet, audit retry, fail-closed ContactRef, PII redaction, `customerfacthttp` split, ciphertext version envelope).
- `backend/rag-core` (submodule added 2026-08-05): `feature/AI-38-workspace-namespace` ← `AI-39-ingest-pipeline` ← `AI-41-retrieval-api` ← `AI-43-kb-editor` (fixes landed on the AI-43 branch: job lease/reaper, per-doc serialization, outbox atomicity, tombstone check). Milvus kept, NOT pgvector.
- `frontend/ai-chat-admin-frontend` (submodule added 2026-08-06, was an empty repo): main ← `feature/AI-92-app-shell` ← `AI-94-company-workspace-mgmt` ← `AI-96-kb-editor` ← `AI-101-conversation-inbox` ← **`fix/admin-review-fixes`** (CRITICAL: logout never revoked the Kratos session; dead `no-dom-in-core` CI rule; error-as-empty-state ×6; 409 UX). Stack: React 18 + Vite + pnpm/Turborepo, ports-and-adapters, DS via `@lit-labs/react`.
- `frontend/sellsuki-invitation`: `feature/AI-21-invitation-i18n-ux` (on top of open MR!16/!17) + `backend/i18n-management-backend` `feature/AI-21-invite-manage-i18n-seeds` (migration 0028, 34 keys).
- `backend/sellsuki-central-control-backend`: `fix/invite-revoke-ownership-guard` — CRITICAL cross-company invite revoke; `backend/sellsuki-role-and-permission-management-backend`: `fix/invite-revoke-scope-check` (defense in depth, optional proto `scope` field).
- AI-36 RAG spike deliverable: `docs/ai-rag-inventory-recommendation.md` (committed to monorepo).
- SRE handoffs (not code): `docs/vault-gateway-handoff.md` (AI-18), `docs/observability-handoff.md` (AI-20), WS ingress timeout (AI-33). See [[project-ai-platform-deploy-gating]].
