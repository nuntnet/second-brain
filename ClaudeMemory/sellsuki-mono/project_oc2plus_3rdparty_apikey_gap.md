---
name: project_oc2plus_3rdparty_apikey_gap
description: OC-2275 rewritten (2026-07-21) — 3rd-party API-key integration (member sync/campaign/point) gap analysis; collides with OC-4322 (duplicate auth mechanism) and OC-4294 (shared AdjustPoint use case)
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-07-21T13:33:34.938Z
---

**OC-2275** (Epic, created 2024-01-11 as an empty stub with 4 stale UI-mockup child stories never implemented) rewritten 2026-07-21 after verifying actual code — not the 2024 spec — by shallow-cloning the real repos into scratchpad (none of these are monorepo submodules).

**Real API-key infra already exists** in `oc2plus-line-crm-service-3rdparty-api` (separate GitLab repo, not a submodule): `X-Api-Key`/`X-Api-Secret` header → `CRMApiKey{ID, CompanyID, Scopes[]}` (`src/use_case/auth_apikey.go:11`), `api_key` table has `company_id, key, secret, scopes[]` — **but only 3 scopes enforced anywhere**: `event.read`, `campaign.read` (paired with event.read only), `campaign.redeem.event` (`src/use_case/model/api_key.go:6-8`). Only 4 endpoints total: `GET /event`, `POST /campaign/event/inquiry`, `POST /campaign/event/confirm`, `GET /auth/whoami`. **No `CreateApiKey` use case exists anywhere** — keys are inserted by hand via SQL; DB table has no `name`/`expires_at`/`revoked_at` columns.

**Gaps found (all confirmed ❌/⚠️ against real code, not assumed):** contacts/member sync via API key (member-api is 100% LINE-identity-coupled, no company-scoped bulk CRUD), full campaign read (only "event" list exists, no campaign object w/ reward), campaign write/post (campaign authoring lives only in backoffice-api, session-auth, and 3rdparty-api's own README says campaign authoring is explicitly out-of-scope/delegated), point redeem-for-reward via 3rd-party, point manual adjust (doesn't exist in ANY service yet).

**Two collision risks flagged in the rewritten epic — check before implementing either:**
- **OC-4322** ("Customer Profile API — external query") is about to design a brand-new service-to-service auth mechanism from scratch, its description literally says "ยังไม่มี pattern การยืนยันตัวตนระหว่าง service กับ service" — **that's stale/wrong**, the API-key-per-company mechanism above already exists. Posted a comment on OC-4322 (2026-07-21) resolving my own earlier 2026-07-04 PO-review blocker on it, pointing it at the existing mechanism.
- **OC-4294** ("Marketer อยาก adjust point ด้วยมือ") will build the first `AdjustPoint/GrantPoint/DeductPoint` use case in the whole system (admin-session consumer). OC-2275's point.adjust scope must reuse that use case with a different entry point (API key), not build a parallel one.

**Final scope list locked in OC-2275 (2026-07-21):** `event.read`, `campaign.read`, `campaign.redeem.event`, `member.read`, `member.manage`, `point.adjust`, `point.redeem` — explicitly cut `campaign.manage`/write (WS4, too risky — campaign authoring stays session-auth-only in backoffice-api).

**4 stale 2024 child stories rewritten 2026-07-21 to match this scope + real schema:**
- **OC-2273** (create key) — now the canonical/only create-flow card. Flags: migration needed for `name`/`expires_at`/`created_by` columns on `api_key` table (repo `oc2plus-line-crm-service-3rdparty-api`), no `CreateApiKey` use case exists yet, secret currently stored plain-text (flagged MUST-FIX before merge — needs hash/encrypt + crypto-random ≥32 bytes, not bare UUID).
- **OC-2269** (list page) — blocked-by OC-2273's migration; revoke = soft-delete via `deleted_at` (column exists, but no `RevokeApiKey` use case/endpoint yet); status is derived (`deleted_at IS NULL` + `expires_at` check), no status enum in schema.
- **OC-2274** (detail view, read-only) — same migration dependency; dropped the old spec's "Description" field (no `description` column in `api_key` table, would need a new migration if ever wanted).
- **OC-2305** — confirmed a pure empty-template duplicate of OC-2273 (same goal, created 12 days apart by the same reporter in Jan 2024). Description now flags it as DUPLICATE/recommend-close pointing at OC-2273; status left untouched (To Do) pending explicit PO confirmation before transitioning/closing.

Links [[project_loyalty_point_cluster]] (point earn/redeem business rules for the campaign engine side) — the two together map most of OC2Plus's loyalty/points surface area.
