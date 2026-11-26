---
name: project-ccs2-no-management-backend-cto-2026-11-26
description: "CTO decision 2026-11-26 — CCS2 (provider-management-frontend) must NOT call management-backend; QMS/provider pages go through space-go provider routes (CTO's SF-PFE-4 on feature/sukispace-integration)"
metadata:
  node_type: memory
  type: project
  originSessionId: 58c6eca6-d6eb-446b-bf6a-a91ba8b63329
  modified: 2026-11-26T15:25:34.847Z
---

CTO ruled (relayed by nunt, 2026-11-26): CCS2 = `frontend/sellsuki-provider-management-frontend` must not go through `backend/management-backend`. The QMS admin pages move to space-go provider routes, as the CTO already did in SF-PFE-4 (324e8a4) on `feature/sukispace-integration`.

**Why:** the CTO's branch had already rebuilt those pages on space-go. Our AI-232 (error wording + company names) and PAT-2735 provider console sat on the management-backend path. That made two architectures for one screen.

**How to apply:**
- New or changed CCS2 screens call space-go provider routes, never management-backend.
- AI-232's behaviour (readable errors, company names instead of ids) must survive on the CTO's pages when that branch lands.
- PAT-2735 (!31/!32) must follow the same direction.

Related: [[project_sukispace_subscribe_only_on_integration_branch_2026_11_20]]
