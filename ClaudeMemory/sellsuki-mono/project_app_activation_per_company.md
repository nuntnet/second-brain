---
name: project_app_activation_per_company
description: "PO 2026-09-27: every app must be explicitly activated per company (within its provider); OC2Plus has none today → OC-4627; company is created 3 ways and none records app usage or plan"
metadata:
  type: project
---

PO decision 2026-09-27: **every app needs explicit per-company activation**, inside the company's own provider. Chat already works this way (CCS `ProviderProvisionChatWorkspace`/`CompanyProvisionChatWorkspace`, gated by `CHAT_CORE_ENABLED_PROVIDER_CODES`); BOLA workspaces are created explicitly in CCS3 (BOLA-310). OC2Plus had no activation — card **OC-4627** adds a per-company activation record + one idempotent endpoint that every entry path uses.

Company creation paths (none records "uses OC2Plus", none assigns a plan or capability): ① Patona seller center → management-backend → CCS `POST /v1/user/company` · ② OC2Plus "สร้างกิจการ" → backoffice-api `CreateCompany` (theme + BOLA bind with owner) · ③ CCS2 → `ProviderCreateCompany` (provider also gets Owner role, PAT-2448). CCS `genericCreateCompany` never creates a BOLA workspace. Without an activation record the backoffice sweep bound every same-provider company with no owner.

Vocabulary the user wants kept separate: **company** (CCS) vs **activated in app X** (the app's own capability record, AI-285 reads it) vs **commercial plan** (management-backend + BOLA-227, not built; later it will call the same activation). OC-4204 (Done) specified "workspace only on activate" but code did the opposite.

**How to apply:** OC-4627 must land in an environment before Patona companies are moved into `sellsuki` there. See [[project_provider_is_whitelabel_tenant]], [[reference_bola_sweep_binds_without_an_owner]].
