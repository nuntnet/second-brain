---
name: project-bundle-in-catalog
description: "Product \"bundle/set\" lives in catalog-service (multi-catalog), not pis-api"
metadata: 
  node_type: memory
  type: project
  originSessionId: 9e057444-0352-41d3-9463-fb0dd304ba9a
---

The "bundle / set product" feature (e.g. กระเช้า/gift set) is already shipped in the
**product_catalog** area, NOT pis-api:
- `backend/catalog-service` — `CatalogTypeMulti` (single vs multi catalog); a multi-catalog IS the bundle.
- `frontend/catalog-frontend` — `src/components/multipleCatalog/CatalogBundle.vue` (up to 100 items).

The `feat/pis-bundle-product-type` branch in `backend/pis-api` is a **separate, overlapping**
implementation of the same concept — likely a duplicate. Do not treat bundle as a PIS gap.

**Why:** gap-analysis once read "catalog has single/multi but not bundles" and a pis-api bundle
branch existed, which misleads you into thinking bundle is an unbuilt PIS feature. It is built — in catalog.
**How to apply:** When asked about bundle/set products, point to catalog-service multi-catalog first.
Open question still: does multi-catalog support combined/custom bundle *pricing*, or only grouping? See [[project_merchant_portal]].
