---
name: project-sukispace-marketplace-ais-vision
description: "Product intent (stated 2026-10-06) for SukiSpace/space-go as multi-seller marketplace on Patona OMS, with \"Sellsuki Shop\" and AIS carrier-billing partnership"
metadata:
  node_type: memory
  type: project
  originSessionId: 9043b5b7-a828-4cca-b908-2d5347a98fbf
  modified: 2026-10-06T06:15:00.675Z
---

On 2026-10-06 the user opened research and planning for space-go (SukiSpace) as a **marketplace website**:

- Orders land in **Patona OMS**; sellers are companies created in Patona who **publish** products to SukiSpace. Product types: physical, service, digital. Fulfillment differs per type and is handled by Patona.
- Sellsuki runs its own shop, **"sellsuki shop"**, selling OC2Plus, Patona and BOLA subscriptions.
- **AIS partnership (Thai telco):** the customer pays through their monthly AIS bill, and AIS remits revenue to Sellsuki monthly. After the customer pays AIS, an order must be created in the sellsuki shop and the buyer company's quota/capability opened, as a capability added to a plan defined in provider management (CCS2).
- Open question the user posed: is the seller the **company** or the **provider (sellsuki)**, given Patona seller center is the OMS?

Findings at that date (prior art): no card, doc or code for multi-seller, AIS or carrier billing. Roadmap Theme 6 has it as 0% with no cards. space-go is single-tenant (`PIS_REF_ID=sellsuki.company:sukispace`). QMS plan to quota works, but there is no capability layer (P3 uncarded), and no payment-to-activation wiring (PAT-2493 is design only). Jira was not searched.

**Why:** a cross-team, multi-repo initiative that is invisible from the code.
**How to apply:** treat it as net-new; reference the roadmap slugs `sukispace-patona-sale-channel` and `sukispace-digital-tech-product-focus` and `docs/analysis/plan-capability-quota-map.md` instead of re-describing them. Search Jira across all statuses before writing a card (shipping.md §11). See [[project-ccs-bola-provisioning-unwired]].
