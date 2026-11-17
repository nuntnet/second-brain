---
name: reference-providers-patona-poshmedica-company-pickers
description: "Production has two CCS providers (patona 309 companies, poshmedica 74, no \"sellsuki\"); patona is being renamed to sellsuki (PAT-1529); each app's company picker filters by its own provider, so lists differ by design"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-17T17:24:12.950Z
---

Read-only check on production CCS DB, 2026-11-17: `companies.provider_code` = `patona` 309, `poshmedica` 74. There is NO `sellsuki` provider yet. nunt: "provider = sellsuki (ที่กำลังจะ rename จาก patona)" — card PAT-1529 renames patona → sellsuki in CCS/rps/Keto. Until it runs, anything that means "the Sellsuki provider" must use `patona` (AI-466 sets `CCS_COMPANY_CREATE_PROVIDER_CODE=patona` with a PAT-1529 note).

Why company pickers disagree (nunt asked 2026-11-17):
- CCS3 `admin.sellsuki.com`: CCS `GET /user/company?provider_code=<VITE_PROVIDER_CODE>` → only patona companies.
- OC2Plus `crm.oc2.plus`: backoffice-api `PROVIDER_CODE=poshmedica` (values-production.yml:197) → only poshmedica companies.
- Chat console: chat-core `/v1/me/companies`, no provider filter → every company with a company role holding chat manage, or workspace access.

**How to apply:** when a list "is missing companies", check the provider of each company first. The rename also has to cover QMS plans per provider, CCS3 `VITE_PROVIDER_CODE`, CCS `CHAT_CORE_ENABLED_PROVIDER_CODES`, and AI-466's env value. Related: [[project-ai-chat-prod-onboarding-state-2026-11-17]].
