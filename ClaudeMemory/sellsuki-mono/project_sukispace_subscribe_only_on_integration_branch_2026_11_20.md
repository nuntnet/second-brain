---
name: project_sukispace_subscribe_only_on_integration_branch_2026_11_20
description: "SukiSpace self-subscribe (SG-10/11/53/82) exists only on feature/sukispace-integration in space-go + space-storefront, not main; sells Patona/Akita plans, not BOLA/AI chat/OC2Plus"
metadata:
  node_type: memory
  type: project
  originSessionId: 58c6eca6-d6eb-446b-bf6a-a91ba8b63329
  modified: 2026-11-20T06:21:43.535Z
---

As of 2026-11-20 the self-service "company subscribes to a plan" flow (space-go `/subscriptions/*`, storefront `SellerPackages`/`SubscribeConfirm`/`/my-contracts`) lives only on `origin/feature/sukispace-integration` (space-go ~243 commits ahead of main, storefront ~188). No MR, never run on dev/staging, `SUKISPACE_SUKIPAY_ENABLED=false` everywhere. Jira shows SG-9/11/82 epics "Done" anyway.

The AI chat console upgrade button (AI-468) goes to a static "coming soon" `/upgrade` page and is not wired to SukiSpace. No BOLA / AI chatbot / OC2Plus plans are mapped in the subscribe flow.

**Why:** Jira status says done while the code is branch-only — easy to plan on top of something that is not on main.
**How to apply:** before planning self-subscribe work, check main vs that branch; treat SG "Done" as branch-done. Related: [[project_oc2plus_saas_plan_epic_oc4763_decisions_2026_10_16]]
