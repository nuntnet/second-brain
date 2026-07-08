---
name: reference_oc2plus_member_frontend
description: "OC2Plus customer member-registration LIFF frontend = existing repo oc2plus-linecrm-frontend-member (GitLab, 2yr); OC-4245 belongs there, not a new repo"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
---

The customer-facing OC2Plus **member registration LIFF frontend** already exists on GitLab as `git@gitlab.sellsuki.com:sellsuki/oc2plus/line-crm/frontend/oc2plus-linecrm-frontend-member.git` — 2+ years old, active. Branches confirm it IS the LINE member-register app: `feature/oc-2396-member-register-line-account`, `enhance/oc-2782-change-text-message-register-line`, `bug/oc-2429-check-member-activation`, `enhance/posh`, `develop`.

⚠️ It is NOT added as a submodule in this monorepo by default — `frontend/` only had `oc2plus-linecrm-frontend-backoffice` (the Vue3 admin). **Checking local monorepo submodules is NOT enough to conclude "no customer frontend exists" — always `git ls-remote` the GitLab group too.**

**OC-4245 (membership register/enrich page) belongs in this existing `-member` repo** (likely an enhancement of the existing register-line flow, wiring it to the new OC-4289 Membership API + domain boundary), NOT a greenfield app.

**OC2Plus line-crm actually has THREE active backends (discovered incrementally — monorepo only had one):**
- `oc2plus-line-crm-service-3rdparty-api` — **the REAL campaign engine**: evaluates campaigns + awards points (`src/use_case/campaign_transaction.go` — calculate→prepare→commit chain, event inquiry/confirm OpenAPI with API keys). Backoffice-api only has campaign CRUD; it never awards points. NOT a monorepo submodule. Campaign facts (verified 2026-07-08): condition types = code/point/**event** (1 event per campaign, merchant-created `EV-xxx`, no system "purchase" event seeded); reward point = **fixed Quantity int64 only** — no formula/amount/conversion-rate anywhere in the domain; campaigns with overlapping date ranges can be created freely (zero overlap validation, no priority/exclusive concept — confirm is per-campaign so caller chooses). OC-4295 (points by purchase) was rewritten to: purchase system event + campaign-level "every X baht = Y point" config translated to quantity=floor(amount/X) through the existing engine path + overlap policy v1 = award ALL eligible campaigns + creation-time warning.
- ⚠️ GitLab repo named `oc2plus-line-crm-service-campaign-engine` is an **EMPTY SHELL** (README only, dead since 2024-10) — don't be fooled by the name; the engine lives in 3rdparty-api.

**Plus the TWO backends found earlier (neither was a monorepo submodule):**
- `oc2plus-line-crm-service-backoffice-api` — admin/backoffice API (HTTP 8089), the only one in the monorepo. Has its own DB.
- `oc2plus-line-crm-service-member-api` — **customer-facing member API** = `api.member.oc2.plus` (dev `api.member.dev-th.oc2.plus`). This is what the `oc2plus-linecrm-frontend-member` frontend calls (`VITE_SERVICE_MEMBER_BASE_URL`). Branches: develop, feature/oc-2313-login, enhance/oc-3426-change-endpoint-management-backend-to-ccs, etc. Existing member-create flow: frontend sends `line_access_token` → member-api creates member (LINE-coupled).

⚠️ **OC-4289 (customer Membership API POST /members) belongs in `oc2plus-line-crm-service-member-api`, NOT backoffice-api.** I built it in backoffice-api by mistake (same root cause — only checked local submodules, didn't ls-remote the GitLab backend group). Needs re-homing to member-api + reconciling with its existing member-create flow (which is LINE-coupled vs OC-4289's LINE-agnostic design — a real design decision). BOLA-264 (bola-backend) is correctly placed.

**Lesson (recurring 3x this session):** before assigning a card's repo home, `git ls-remote` the GitLab group (`sellsuki/oc2plus/line-crm/{frontend,backend}/...`) — the monorepo only contains a subset of repos as submodules.

**Audit conclusion (2026-06-26, OC-4200 chain):** line-crm group = exactly 2 backend + 2 frontend + 1 testing repo. Only **OC-4289 is misplaced** (built in backoffice-api, card says member-api). OC-4207 ✅, OC-4267 ✅ (both correctly in backoffice-api — admin/schema domain; OC-4267 owns `oc2plus_bola_bindings`), BOLA-264 ✅, OC-4245 ✅ (frontend-member). OC-4241/4242/4290 → backoffice-api + frontend-backoffice (admin); OC-4263/4291 → provider-mgmt-frontend + CCS. **member tables (`member`,`member_identity`) live in a SHARED DB** owned by external module `line-crm/backend/entity` v1.7.3 — both backoffice-api (raw SQL) and member-api (goqu) connect to the same `POSTGRES_CRM_DB_NAME`. The new tables OC-4289/4207 created (`member_bola_follower_link`,`member_line_snapshot`,`otp_sessions`) sit in that shared DB. **member-api existing `POST /v1/member` is LINE-coupled** (requires line_access_token → calls api.line.me for profile, keys identity on LINE user id, NO OTP, NO BOLA link) — 180° opposite to OC-4289's LINE-agnostic design. Recommended fix = move OC-4289 to member-api as a NEW endpoint `POST /v1/members` (LINE-agnostic) alongside existing `/v1/member` (don't break login); port OTP/SMS/bola-contact/link-table code + migrations from backoffice-api; binding_status (OC-4267, stays in backoffice-api) becomes a cross-service read.

Mistake made 2026-06-26: I scaffolded a NEW repo `frontend/oc2plus-linecrm-frontend-liff` (React18/Vite/@line/liff, port 5183) + wired the monorepo (commit `bfb944d` on `feat/oc-4200-member-follower`) believing no customer frontend existed — because I only checked local submodules. The `-liff` GitLab project never existed (404). Corrected by discarding the scaffold + adding the real `-member` repo as submodule. See [[project_oc_bola_domain_boundary]].
