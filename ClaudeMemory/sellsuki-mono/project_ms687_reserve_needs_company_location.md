---
name: project-ms687-reserve-needs-company-location
description: MS-687 (OMS↔inventory reserve/deduct) blocked — reserve/pre-order needs company + location
metadata: 
  node_type: memory
  type: project
  originSessionId: 9e057444-0352-41d3-9463-fb0dd304ba9a
---

MS-687 (`[Tech] OMS ↔ Inventory: reserve / deduct / return stock`) is **BLOCKED**.

inventory stock is keyed **per (company × location × variant)** — so every op
(reserve / deduct / return / **pre-order**) must pass `ref_company_id` + `ref_location_id`.
Dev feedback (2026-06-18): the **company-location flow isn't complete**, and OMS orders
don't currently carry company/location in the shape inventory needs.

**Open question (must resolve before implementing MS-687):** how does an order resolve its
location? — default location per company / mapping from POS store / passed on the order at
create time; and which location does a **pre-order** use (goods not in stock yet)?

**How to apply:** MS-687 is blocked-by company-location setup (MS-993/994/1013 location CRUD,
MS-1073 create-from-kafka). Don't schedule MS-687 implementation until location resolution is
answered. Related: OMS↔inventory is currently a dummy repo (no real reserve). See [[project_bundle_in_catalog]].
