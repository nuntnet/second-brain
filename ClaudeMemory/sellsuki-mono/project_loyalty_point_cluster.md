---
name: project_loyalty_point_cluster
description: "Loyalty/point card cluster (OC-4295/4297/4335/4339) — designs locked 2026-07-08/09: purchase event + conversion, welcome point, receipt-QR pull model, clawback"
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

**Loyalty point cluster under epic OC-2743 (Point Budgeting Management) — designs locked with user 2026-07-08/09.** Engine facts (3rdparty-api = real engine, campaign-engine repo = empty shell, reward = fixed Quantity only) are in [[reference_oc2plus_member_frontend]].

The 5-card map and key locked decisions:
- **OC-4295** (point ตามยอดซื้อ): purchase system event under existing campaign engine + per-campaign conversion config "ทุกๆ X บาท = Y point" → `quantity = floor(amount/X)` through existing engine path; rounding = floor per bill; overlap policy v1 = **award ALL eligible campaigns** (no priority/exclusive; warning at create, never block); rate field lives in shared entity module → version bump BO + 3rdparty together.
- **OC-4297** (welcome point): register event, fixed amount (NO conversion needed); gap = member creation emits NO event today — must emit member.created at use-case level in TWO places (member_liff.go:174 new path, member.go:138 old path; repo layer can't distinguish new/existing due to ON CONFLICT DO NOTHING); LimitUsagePerMember=1 is TOCTOU-racy → needs DB unique constraint (member_id, campaign_id). QR-receipt = Layer B split out.
- **OC-4335** (QR ท้ายใบเสร็จ, 2 โหมด): mode A fixed = reuse existing code-gen lots (works offline, POS batch-pulls codes); mode B by-amount = **pull model per user's proposal**: one-time code bound to order serial only (no amount baked in) → at claim, validate code → call Patona order-lookup API {amount, status, purchased_at} → compute → consume(CAS)+award in one txn. Rate = campaign active at **purchased_at** (decision closed). Fetch-before-consume so OMS downtime never burns the code. Dedup across paths = unique order serial shared with OC-4295 auto-award. Integration: OC2Plus provides code APIs on 3rdparty-api (API-key infra exists), Patona = requester → POS work is a PAT-board card (contract written in OC-4335); open decision: code↔serial binding = async registration (recommended) vs HMAC-signed serial.
- **OC-4339** (clawback): order cancelled/refunded AFTER award → compensating deduction found via order-serial attribution; needs new activity type `revoke` (enum today = earn|redeem|expire only) in shared entity module; open business decisions: negative balance vs floor-0 (recommended: allow negative), partial refund pro-rata, welcome point NOT clawed back.
- **OC-4294** = manual adjust (interim/fallback).

QR inventory (verified by grep 2026-07-08): OC2Plus has NO real QR generation — `outline-qr-code` is just a menu icon for the "โค้ด" feature; BOLA has the pattern to copy (`qrcode.react` client-side rendering of URLs, LONSubscribersPage); payment QR (KBank) in space-go/sukipay is a different domain. QR = URL rendered client-side; never build backend image generation.

Existing code-gen (โค้ด menu) deliberately NOT reused for receipt tokens: pre-generated lots, no amount payload, per-code dedup only, merchant-managed lifecycle vs system-minted per-receipt volume. But "fixed-point QR poster" use case can use existing code-gen today with only a FE render button.
