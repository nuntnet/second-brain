---
name: project_provider_management_prior_art
description: "Provider management (white-label tenant admin) is already specced in Jira PAT-1525/1527/1529/2739/2740/2741 (all To Do/Draft); CCS1=system admin creates/brands providers, CCS2=provider admin 'My Provider'; provider presets are NOT reconciled and refresh-role likely 409s"
metadata:
  node_type: memory
  type: project
  originSessionId: 013c2780-b679-4183-8772-822df5141712
  modified: 2026-09-29T17:01:47.243Z
---

Found 2026-09-30 while the user asked for a feature design of `system.sellsuki.local/provider`.

- **Specs already exist in Jira, none built:** PAT-1525 (CCS1 provider detail: profile/domains/logo+theme, `PUT /v1/admin/provider/{code}`, `provider_domain` table, perm `sellsuki.provider.branding.manage` on `sellsuki.system`), PAT-1527 (Provider Owner RBAC + system-admin perm group), PAT-2739 (`provider_app` — apps a provider offers), PAT-2740/2741 (CCS2 "My Provider" + provider members), PAT-1529 (`patona`→`sellsuki` migration), OC-4633 (quota templates). Epic PAT-1511. Check these before writing any new provider card.
- **Split by audience:** CCS1 = `sellsuki-system-management-frontend` (system admin, cross-provider). CCS2 = `sellsuki-provider-management-frontend` (one build per provider via `VITE_PROVIDER_CODE`; companies, QMS, plans, AI model settings). CCS3 = company.
- **Built today:** list + create only (system-mgmt `/provider`, develop only). CCS `providers` table has name/code(`company_code` col)/logo/full_logo/address/tax — no status, timestamps, domains, theme, apps. No provider delete/suspend, no provider member API.
- **RBAC traps (from code reading, not run):** AI-252 reconciler covers company presets only — `ProviderPermissions` (21 codes) never reach existing providers. `POST /v1/admin/provider/{code}/refresh-role` calls rps `CreateRole` (plain INSERT, `uk_roles_name_owner`) so an existing provider likely gets 409; `is_system` blocks `UpdateRole`. Provider create is not transactional (row before role).
- Local `.env.dev` `VITE_CCS_URL` lacks `/v1` → provider list 404s locally.

**PO decision 2026-09-30:** "จัดการ rps" = **edit permissions and role presets directly from the CCS1 UI** (not view-only — I had recommended view + controlled actions; overruled, don't re-argue it). CCS1 provider console scope also includes: quota, pricing plan, usage logs, customer support, configuration, portal app registry management, warehouse management, Shipmunk enable.

**How to apply:** start from these cards rather than a parallel design; the open product work is provider lifecycle/status, provider-level preset propagation, and an rps console. Related: [[project_provider_is_whitelabel_tenant]], [[reference_preset_copy_vs_shared_role]], [[reference_dev_has_no_provider_owner_role_or_system_admin]], [[project_app_activation_per_company]].
