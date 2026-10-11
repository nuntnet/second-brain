---
name: project-dev-onboarding-dead-ends-2026-10-11
description: "Two measured defects that stop a freshly registered user on dev-th: (1) admin FE creates companies through the PROVIDER route (403 for a normal user) while a user route POST /v1/user/company exists; (2) CCS portal-app-registry dev seed points the Oc2plus tile at crm.dev-th.oc2.plus which does not resolve (crm.dev.oc2.plus does). Found 2026-10-11, not fixed."
metadata:
  node_type: memory
  type: project
  originSessionId: 5077123b-d938-4a36-b54b-335561e7b9a9
  modified: 2026-10-11T15:08:33.054Z
---

Found while trying to give the verifier agent a dev session (identity `26861216-…`, registered on accounts.dev-th, no company).

**1. No company → dead end in admin (admin.dev-th.sellsuki.com).** `frontend/sellsuki-company-management-frontend` `CompanyCreate` calls `createCompany` which posts to `/provider/<code>/company` (the unit test pins `expect(request.url).toBe('/provider/test-provider/company')`). CCS `ProviderCreateCompany` first runs `checkPermission(… SellsukiProvider, c.ProviderCode, PermissionSellsukiCompanyCreate)` → a normal user gets `403 permission_denied` (seen in the pod log: `POST /v1/provider/patona/company` → 403). CCS also has `POST /v1/user/company` (`PostUserCompany` → `UserCreateCompany`), whose first 32 lines only validate phone, tax id and postal code (I did not read the rest for a later permission check). Registration itself creates no company (kratos-ui-go; the auto-create path exists only on `feature/sukispace-integration`). So a self-registered user can enter admin and cannot create a company.

**2. Portal tile host wrong on dev.** `backend/central-configuration-system/cmd/migration/migrations/0003_seed_portal_app_registry_config/data/config.development.json:121` seeds the Oc2plus `deep_link.url` as `https://crm.dev-th.oc2.plus/`. DNS 2026-10-11: `crm.dev-th.oc2.plus` NXDOMAIN; `crm.dev.oc2.plus` resolves (CloudFront). Staging seed `crm.staging-th.oc2.plus` and production `crm.oc2.plus` both resolve. Config migrations are applied by hand per environment, so fixing the file does not change dev by itself. The admin FE tests also hard-code the dev-th host in fixtures.

**How to apply:** before telling anyone "register then create your company in admin", remember it 403s; the working paths today are an invitation into an existing company or a provider-level grant. If asked to fix: (1) FE should call the user route for non-provider users or CCS should grant the permission; (2) one-line seed fix plus applying the config on dev. Related: [[project-ccs-bola-provisioning-unwired]], [[reference-central-config-develop-lacks-main-seeds]], [[project-invite-multiapp-chain]].
