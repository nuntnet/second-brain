---
name: reference-ccs2-staging-index-html-cached-dev-mgmt-401
description: "CCS2 (provider-management-frontend): staging index.html has no Cache-Control so browsers serve the old bundle after deploy; CCS2 dev calls management on api.dev-th.patona.online so every call is 401"
metadata:
  type: reference
---

Found 2026-11-03/04:
- **Staging cache:** https://ccs.staging.sellsuki.com serves index.html with Last-Modified and no Cache-Control → heuristic browser cache; after a deploy a plain reload kept `index-<old>.js` (`performance.getEntriesByType('navigation')[0].deliveryType === 'cache'`). Verify a deploy by loading the page with `?v=<timestamp>` and reading `document.scripts` for the bundle hash. Real fix = `Cache-Control: no-cache` on index.html in the image's nginx.
- **Dev unusable for QMS pages:** `.env.development` sets `VITE_MGMT_API_URL=https://api.dev-th.patona.online/management`, a different domain from the Kratos cookie → 401 `unauthenticated` on every management call (`/v1/plans`, `/v1/qms/quotas`, `/v1/user/profile`). Test CCS2 QMS changes on staging.
- `ai_provider_admin` lacks `sellsuki.company.view` → `GET /v1/company/:id` 403; the provider company search matches code/name, not id.
