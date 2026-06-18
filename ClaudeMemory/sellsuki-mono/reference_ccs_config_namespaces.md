---
name: reference-ccs-config-namespaces
description: "central-configuration-system live namespaces, their scopes, and all config keys"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 56602b5c-05ea-4d80-b7ba-3129b2ee78d0
---

# CCS (central-configuration-system) Config Namespaces

Port 8085. Dynamic schema-driven store — no hardcoded keys in code, schemas registered at runtime.

## Active namespaces (as of 2026-06-18)

### `patona-seller-center-user`
- Scope: `?userId=:userId` (user-level)
- Keys: `uiTheme`, `language`, `selectedCompanyId`, `selectedStoreId`, `defaultCurrency`, `notifications` (newOrderEmail, lowStockWarningApp, systemUpdatesEmail)
- Caller: sellercenter-frontend → management-backend proxy → CCS

### `patona-seller-center-store`
- Scope: `?location=:storeId` (store-level)
- Keys: `timeoutSeconds` (int) — order expiry timeout in seconds
- Write endpoint: `PUT /v1/configuration/store/patona-seller-center-store?location=:storeId` via management-backend
- Permission required: `PermissionPatonaStoreOrderExpiryEdit`
- Caller: sellercenter-frontend → management-backend proxy

### `sellsuki-global-user`
- Scope: `?userId=:userId` (user-level)
- Keys: `language`, `selectedCompanyId`
- Caller: sellsuki-company-management-frontend → CCS directly via `VITE_CENTRAL_CONFIGURATION_SYSTEM_URL`

## Missing (needs new namespace)

`patona-provider-company` — NOT YET CREATED. Proposed schema for provider-management-frontend:
```json
{
  "defaultStoreId":   { "type": "string", "default": "" },
  "defaultLanguage":  { "type": "string", "default": "th" },
  "defaultCurrency":  { "type": "string", "default": "THB" }
}
```
Scope: `?location=:companyId`

**Why:** defaultStore / defaultLanguage / defaultCurrency are currently only user-level preferences, not company-level. Provider management needs company-scoped config.

**How to apply:** When building provider-management-frontend company settings, Phase 3+4 of PAT-2452 depend on this namespace being created first (backend task).
