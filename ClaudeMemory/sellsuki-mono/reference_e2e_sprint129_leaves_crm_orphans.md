---
name: reference_e2e_sprint129_leaves_crm_orphans
description: "OC2Plus e2e (feature/sprint-129) creates a company, presses \"เริ่มใช้\", then deletes it from CCS only — CRM activation/binding/theme rows stay behind; CCS answered 500 for absent companies (fixed in !400/!401) so the binding sweep retried them forever and staging bind_failed/preparing numbers are mostly test junk"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7fb71c5f-6e71-4687-b858-9caa035598f1
  modified: 2026-10-10T15:03:59.163Z
---

Found 2026-10-10 while verifying OC-4591 on staging.

- **What the test does.** `testing/oc2plus-line-crm-e2e-playwright` branch `feature/sprint-129`, flow `sign-in.resource.ts` → `dpa-consent.page.ts`:
  - clicks `dpa.start.cta`, which calls `POST /backoffice/v1/company/{id}/activate` for every new company;
  - teardown `clearCompanyInformation` runs `DELETE FROM public.companies` in the CCS DB (`repository/db-staging-patona-management.ts`).
- **What it leaves behind.** Nothing on that branch deletes `oc2plus_company_activation`. The binding is deleted only in some BOLA fixtures. So `oc2plus_company_activation`, `oc2plus_bola_bindings` and `company_theme` rows stay in the CRM.
- **How to spot the rows.**
  - binding created in the same second as an activation with `source=activate`
  - a distinct activating user per company, 0 members
  - bursts of 50–90 rows per hour
- **The CCS side.** CCS `GetCompanyById` wrapped errors with `%v`, so an absent company answered 500. Fixed by mapping it to `model.ErrNotFound` → 404 / gRPC NotFound in CCS !400 (develop) and !401 (main), both opened 2026-10-10 at night and not yet merged.
- **2026-10-10 cleanup.** 394 companies × (binding, activation, theme) were deleted on staging with the user's OK; recorded in `docs/release/environments.yaml`.
- **It will happen again** every time that e2e runs, until QA's teardown deletes the CRM rows too.
- **Effect on metrics.** Staging `bind_failed` and backfill `cannot_grant` (2,585 owners missing from Kratos) are mostly this kind of test data. Don't read them as product health. Alerts built on AC-10 need a filter.

Related: [[project_app_activation_per_company]] · [[reference_oc2plus_e2e_playwright_repo]]
