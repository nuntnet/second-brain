---
name: reference_oc2plus_3rdparty_api_two_surfaces
description: "oc2plus-line-crm-service-3rdparty-api serves TWO principals — /v1/* member session (has the consent gate) and /v2/openapi/* company API key for POS (no consent gate, scopes only); nothing but the code says which"
metadata:
  node_type: memory
  type: reference
---

Established 2026-09-22 while judging OC-4559.

| | `/v1/*` | `/v2/openapi/*` |
|---|---|---|
| principal | member session | company API key |
| headers | `X-User-Id` + `X-Session-Id` + `X-Integration-Id` | client sends `X-Api-Key` + `X-Api-Secret`; the gateway/whoami exchange then sets `X-Api-Key-Id` + `X-Company-Id` + `X-Api-Scope` |
| header reader | `helper.GetIdentityFromHeader` | `helper.GetApiKeyIdentityFromHeader` |
| guard | `checkAuthenAndAuthroize` → **consent gate** | `RequireApiKeyScope` → **scopes only, no consent** |
| routes | campaign_v1 · consent_v1 · me_v1 · me_coupon_v1 · order_v1 · internal_v1_member_card | campaign_v2 · code_v2 · coupon_v2 · event_v2 · member_card_v2 · member_v2 · point_v2 |
| consumer | customer app (LIFF / web / custom) | **POS and system integrations** |

They share no method. v2 calls `ApiKeyCodeInquiry` / `ApiKeyCodeRedemption` /
`MemberPointSummary`; v1 calls `CodeInquiry` / `CodeRedemption` /
`GetSummaryPointExpire`. All 11 methods that call `checkAuthenAndAuthroize` take
`oc2plus_identity.Identity` + `SessionInfo`; none takes `model.CRMApiKey`.

**Why it matters:** a POS cannot hold a member session and has no screen on
which a customer could accept a consent document — not even at the counter
(`member_card_v2`). So any work on the consent gate is **v1 only**; adding it to
v2 "for consistency" breaks every POS integration. That is now written on
OC-4559.

**Discoverability is the weak point.** The only durable statement of the split
is `README.md` lines 43–44 and 59 ("session-authenticated V1" vs
"API-key-authenticated V2"), and:

- it never says the consent gate sits on v1 only — the single fact that decides
  the scope of OC-4559;
- it was written 2026-03-23 and never touched, so it still describes v2 as
  "events and campaign redemptions" while point (OC-4473), code (OC-4472),
  member, member_card and coupon all landed Aug–Sep 2026.

The header names in the README are **correct**, though — checked, not assumed.
