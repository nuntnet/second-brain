---
name: project-provider-frontend-backend-routing
description: "provider-management-frontend uses CCS backend (port 8092), NOT management-backend; these two backends serve different audiences"
metadata: 
  node_type: memory
  type: project
  originSessionId: 56602b5c-05ea-4d80-b7ba-3129b2ee78d0
---

# Provider vs Seller Backend Routing

Two backends are easily confused — they have overlapping APIs but serve different frontends:

| Backend | Port | Serves | Company API base |
|---------|------|--------|-----------------|
| `sellsuki-central-control-backend` | 8092 | **provider-management-frontend** | `PUT /v1/company/:id` — fields: name, code, logo, phone, phone2, address, taxInfo, tosId (NO email) |
| `management-backend` | 8083 | **sellercenter-frontend** | `PUT /v1/company/:id` — same shape + email field + stores CRUD |

## Key differences

- **Store CRUD**: only in management-backend (`GET/POST /v1/company/:id/store`, `PUT/DELETE /v1/store/:id`)
- **File upload proxy**: only in management-backend (`POST /v1/file/public?refId=`)
- **CCS config proxy**: only in management-backend (`GET/PUT /v1/configuration/:service`, `PUT /v1/configuration/store/:service`)
- **Company email field**: only in management-backend's CompanyData — NOT in sellsuki-central-control-backend

## Why it matters

When building provider-management-frontend features:
- Phase 1 (company profile edit) → sellsuki-central-control-backend ✅ already connected
- Phase 2+ (store CRUD, logo upload, CCS config) → management-backend ⚠️ need to add `VITE_MANAGEMENT_URL` env + new axios client

**How to apply:** Any time provider-management-frontend needs store CRUD, file uploads, or CCS config access, it must call management-backend (add new env + service layer). Don't assume it already has access.
