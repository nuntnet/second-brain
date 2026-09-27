---
name: project_provider_is_whitelabel_tenant
description: "PO 2026-09-27/28: provider = top-level white-label tenant, one deployment per provider (env); sellsuki = ours; 'patona' as provider = mistake (it is an app) → migrate to sellsuki; poshmedica = REAL white-label, stays; prod OC2Plus today is Posh's deployment"
metadata:
  node_type: memory
  type: project
  originSessionId: 0f82b824-5c8b-4490-a5fb-553341a58b63
  modified: 2026-09-27T16:45:18.685Z
---

User (PO/architect) stated on 2026-09-27:

- **Provider = the top-level tenant, designed for future white-label.** Selling the system to someone = opening a new provider for them, isolated from ours.
- **Deployment model: one deployment per provider, checked via env** (`PROVIDER_CODE`). Not one deployment serving many providers.
- **`sellsuki` = our own provider.** Every app we run for our own customers (Patona, OC2Plus, BOLA, Chat, SukiPay) belongs under `sellsuki`.
- **`patona` as a provider code is WRONG** — Patona is an app name. Today it is hardcoded as a provider in sellercenter-frontend (`App.svelte:53`, `management-rest.ts:58`, `.env.*`), company-management-frontend and space-storefront `.env.production`, CCS `cmd/auto_assign_preset_roles/main.go:268`, and as `envDefault:"patona"` in all three OC2Plus Go services.
- **`poshmedica` = a REAL white-label customer — do NOT move it to `sellsuki`** (PO, 2026-09-28; reverses a 2026-09-27 statement that its companies were ours). Evidence: OC-3799 (own prod domains *.poshmedica.co.th, "provider A cannot see provider B"), OC-3362 (Setup Posh Provider), OC-4130 (Docker image for Posh), OC-4143 (DB backup delivery). OC2Plus production backoffice-api runs with `PROVIDER_CODE=poshmedica`, so **the current prod OC2Plus deployment IS Posh's white-label**; our own `sellsuki` OC2Plus in production needs a separate deployment (epic OC-3982 "CRM System Implement for Sellsuki Provider").

**Why:** the code treats provider inconsistently (OC2Plus prod: backoffice=poshmedica, member-api unset→patona, 3rdparty=poshmedica but unused; dev/staging all default patona). Without this decision every "which companies does app X see" question has no principled answer.

**How to apply:** never propose crossing providers to let a company use another app — a company activates apps *within* its provider. Per-company app activation is the separate, lower layer (see [[project_line_setup_dual_surface_direction]]). Before assuming a provider value is right, check it against this decision; `patona` in a provider field is a known defect, not a design. Related: [[reference_envdefault_localhost_masks_missing_config]] (same envDefault anti-pattern), [[project_oc2plus_company_not_store]].
