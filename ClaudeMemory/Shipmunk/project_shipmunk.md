---
name: Shipmunk Project Overview
description: Core context for the Shipmunk 4PL shipping platform project
type: project
---

Shipmunk คือ "Stripe for Shipping" — 4PL platform normalize carriers (Flash, Kerry, J&T, NinjaVan, ThaiPost) เป็น API เดียว

**Why:** สร้าง unified shipping API ให้ OMS/WMS/eCommerce integrate ครั้งเดียวได้ทุก carrier

**How to apply:** ทุก feature ต้องผ่าน normalization layer — ไม่ expose carrier-specific fields ออก API

**Key decisions:**
- Base: boilerplate `git@gitlab.sellsuki.com:sellsuki/share/backend/boilerplate-backend-go.git`
- HTTP: **Fiber** (ไม่ใช่ Chi ตาม spec)
- DB: **GORM** (ไม่ใช่ sqlc)
- Logger: **sellsuki-go-logger**
- Config: **caarlos0/env**
- Structure: `src/entity/`, `src/use_case/`, `src/repository/`, `src/carrier/`, `src/interface/`
- Queue: Asynq (Redis-backed)
- OpenAPI: TypeSpec → oapi-codegen

**Additional requirements (beyond spec):**
1. Carrier Cost Reconciliation — track actual carrier charges (API/email invoice/CSV) vs merchant charges → margin per shipment
2. Wallet Pre-Authorization — hold estimated amount on booking, settle with actual after carrier confirms
3. Return Parcel Charges — returns are charged separately, linked to original shipment

**Carriers (all implemented):**
- Flash Express — REST + webhook + cost API (MVP)
- Kerry Express — REST + polling, no cost API (MVP)
- J&T Express (`jnt`) — webhook + cost API, HMAC-SHA256 (Phase 2, done 2026-03-22)
- NinjaVan (`ninjavan`) — webhook, no cost API, HMAC-SHA256 (Phase 2, done 2026-03-22)
- Thailand Post (`thaipost`) — polling only, no cost API, Thai+English statuses (Phase 2, done 2026-03-22)

**Note:** `cmd/api/main.go` and `cmd/worker/main.go` are gitignored (patterns `api`, `worker` in .gitignore match binary output names). All carrier changes (factories + env vars + seed) exist locally but are not committed. Use `git add -f` to force-track if needed.

**Env vars for Phase 2 carriers:** `JNT_API_KEY`, `JNT_WEBHOOK_SECRET`, `NINJAVAN_API_KEY`, `NINJAVAN_WEBHOOK_SECRET`, `THAIPOST_API_KEY`

---

## Blocking Items Status (as of 2026-03-22)

| # | Item | Status |
|---|------|--------|
| R1 | Ghost shipment on crash | ✅ Done |
| R2 | Topup double-charge | ✅ Done — `payment_intents` checkpoint + `ReconcileTopupPayments` cron (every 30min) |
| R3 | Carrier webhook HMAC | ✅ Done |
| R4 | Tracking idempotency | ✅ Done |
| R5 | WalletSettleHold idempotency | ✅ Done |
| R6 | RetryWebhook O(n) scan | ✅ Done — `GetDeliveryByID` + `GetEndpointByID` O(1) lookups |
| R7 | PollTracking silent failures | ✅ Done |
| R8 | /health endpoint missing | ✅ Done |

## Next Priority (from roadmap.md)

```
M4   unit tests — CI gate ✅ Done — coverage 79.0% (entity/merchant DefaultBillingMode, repository model roundtrip tests wallet/shipment/tracking, carrier HTTP tests all 5 carriers)
R6   ✅ done
M12  drop-off sync job
M13  Flash mchid field
M11  Makefile cleanup
M1   invoice parser (waiting on sample files)
M2   rate limiter
M3   Go SDK
M5   Sentry
M8   dead letter queue
M14  rates caching
```
