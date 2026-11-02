---
name: project-dev-onboarding-dead-ends-2026-10-11
description: "Two blockers that stop a freshly registered user on dev-th (OC-4737). State 2026-10-12: create-company fixed in a frontend branch (not merged, not run on dev); portal tile host fix blocked because no agent identity can edit live CCS config (needs sellsuki.configsystem.config.update)."
metadata:
  node_type: memory
  type: project
---

Found 2026-10-11 while giving the verifier a dev session (identity `26861216-…`, registered on accounts.dev-th, no company). Card ledger: `docs/cards/OC-4737.md` (has the guarded console script). Jira OC-4737.

**1. Create company 403.** Admin FE posted to `/provider/{code}/company` (needs `sellsuki.company.create` on the provider). CCS `POST /v1/user/company` has no permission check, validates provider_code and makes the caller Company Owner, same body. **Fixed 2026-10-12** in `frontend/sellsuki-company-management-frontend` branch `fix/OC-4737-create-company-user-route` (8886c84a), test red then green. NOT merged, NOT run on dev, no MR yet. Open PO question: any signed-in user can now create a company under any existing provider code.

**2. Portal tile host wrong.** Live dev `portal-app-registry` is version 11 and only `oc2plus.deep_link.url` is wrong (`crm.dev-th.oc2.plus` does not resolve; `crm.dev.oc2.plus` does). Seed fix is CCS !155 (open) but the seed never changes a live env. The test identity holds neither `sellsuki.configsystem.config.update` nor `.view` (rps CheckPermission false), a PUT from its session returned 403, nothing changed. Someone with `config.update` must run the script in the ledger; dev has no system admin, so a grant may be needed first. Record version 11 → 12 in `docs/release/environments.yaml` afterwards.

**How to apply:** do not tell anyone "register then create your company in admin" until the FE branch is merged and deployed to dev. The config edit is a person's job, never an agent's (auto mode blocks rps writes and I will not grant myself the right). Related: [[reference-ccs-global-config-permission-gate]], [[reference-dev-has-no-provider-owner-role-or-system-admin]], [[project-ccs-bola-provisioning-unwired]].
