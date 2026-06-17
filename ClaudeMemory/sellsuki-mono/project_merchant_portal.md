---
name: merchant_portal_is_shipmunk
description: The merchant portal is shipmunk-frontend, not sellercenter-frontend
type: project
---

The "merchant portal" refers to `frontend/shipmunk-frontend` (at `shipmunk-merchant.sellsuki.local`, port 3004) — NOT `frontend/sellercenter-frontend`.

**Why:** User corrected this when I added a feature to sellercenter-frontend by mistake. The domain-routing.md maps `shipmunk-merchant.sellsuki.local` → `shipmunk-frontend (merchant)`.

**How to apply:** Whenever the user mentions "merchant portal" in the context of shipmunk, always work in `frontend/shipmunk-frontend/apps/merchant/`. The shipmunk-frontend repo is a Next.js 15 / React 19 monorepo with three apps: merchant (3004), playground (3003), tracking (3005).
