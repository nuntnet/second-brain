---
name: project_qms_ui
description: QMS Admin UI lives in provider-management-frontend; proxy endpoints in management-backend; Epic OC-4257
metadata: 
  node_type: memory
  type: project
  originSessionId: 4c69d7f8-cb2f-4e3b-8a1f-3a646f6c6109
---

QMS Admin UI (Epic OC-4257, 14 stories) targets `frontend/sellsuki-provider-management-frontend` (CCS2, port 5178), NOT system-management-frontend.

**Why:** Provider management is the correct home for quota admin — user explicitly corrected this assumption during D1 of plan-ceo-review.

Backend proxy: all QMS gRPC calls are wrapped as HTTP REST endpoints in `backend/management-backend` (port 8083) — QMS has no REST API of its own (gRPC-only on port 50059).

**How to apply:** When asked about QMS UI work or OC-4257 child stories, default to provider-management-frontend for frontend and management-backend for the HTTP proxy layer. There is no dedicated QMS HTTP gateway.

Key stories: OC-4260 (9 proxy endpoints), OC-4269 (fan-out assign plan list — no ListAssignPlans gRPC endpoint exists), OC-4275 (Robot Framework test suite).
