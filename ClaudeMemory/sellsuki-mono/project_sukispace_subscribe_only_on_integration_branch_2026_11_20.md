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

**Update 2026-11-26 (overlap check for the CTO):** the branch is now space-go 573 / QMS 377 / CCS2 FE 133 / CCS 23 commits ahead of main, still no MR, mostly bot-authored ("SukiSpace Plan 2": seller plans, postpaid, buyer party, data-move tool; CCS2 QMS pages moved from management-backend to space-go provider routes). Trial-merge into main conflicts in QMS assign_plan (proto/server/repo/use case), 14 CCS2 QMS pages, CCS role migrations + main.go; clean in space-go. Duplicated intent: space-go `/v1/admins` identity guard (CTO 300d6bf vs our SG-22 MR !2), QMS usage plan lock (CTO d293102/c3e0a26 vs ours 3020a49/f7615a4), QMS NotFound for no plan (c336f9f vs AI-450 4016c78), ensure-once (4f653a8 vs trial 6a6bd24), QMS page error wording/company names (SF-PFE-ALIGN vs AI-232). CCS2 wave1/2 (PAT-2735, !31/!32) will also conflict.
