---
name: project_oc2plus_member_react_migration
description: "Plan to move oc2plus-linecrm-frontend-member (Vue3/Pinia) to React + frontend-kit clean-arch (assessed 2026-09-08): do kit-agnostic prep now, gate kit binding on MR !1 merge/tag or PAT-2660 done, feed cookie-only AuthPort + shell-liff requirement into PAT-2660"
metadata: 
  node_type: memory
  type: project
  originSessionId: 0f82b824-5c8b-4490-a5fb-553341a58b63
  modified: 2026-09-08T16:50:01.676Z
---

**Ask (2026-09-08, user):** migrate `frontend/oc2plus-linecrm-frontend-member` to (1) frontend-kit, (2) React so it can become PWA / React Native later, (3) clean-architecture boilerplate. I assessed readiness; user then asked "ทยอยทำก่อน หรือรอ spike" and "idea ของ kit ดีกับ project เรามั้ย".

**Verdict given:** not ready to migrate wholesale; ready to prep. Kit's lower layers (Result/taxonomy/treatment, layer gate, orval, keyed http, Intl, typed TH/EN catalog) match recurring bugs in this repo (3 duplicated `errorHandler`s, the "404 slug read as member_not_found" bug, 15 `instanceof XxxApiError` branches, views with 315–457 script lines). Kit's `core/auth|permissions|workspace|flags` are staff-app shaped and irrelevant to a customer app. RN reuse = domain/usecases/stores only, never views (Lit DS).

**Recommendation (user has not yet decided):**
1. Do now, kit-agnostic: E2E harness on web surface (OC-4370 — **repo `oc2plus-customer-app-e2e-playwright` does not exist yet**; member CI unit/e2e are `echo "1 + 1"` stubs; only 9 vitest spec files), pull logic out of the 5 views into pure TS `domain/dto/usecases` returning `Result`, 8-line `call()` boundary, upgrade DS `sellsuki-components@0.7.71` (unscoped) → `@sellsuki-org/sellsuki-components@0.27` (still exports `applyTheme`, `IdbI18nStore`, `ssk-theme-provider/text/image/icon`), prep CI for Node 22 + pnpm (current template `generic-frontend-vue-npm-th`, node:20).
2. Send into PAT-2660 now: `AuthPort` must accept cookie-only session without workspace/permissions; `Identity` must not require `companyId` at boot; claim `shell-liff` for the OC2Plus team as first customer-facing consumer.
3. Wait for kit: adapter-react runtime, core/auth, orval client from member-api `src/interface/fiber_server/spec/v1.yaml` (15 paths), React views. **Gate:** MR !1 merged to `main` + `v0.x` tag on registry, or PAT-2660 Done with cookie-capable AuthPort.
4. Migrate views per route (strangler): `/:slug/login` → `/:slug/register` → `/:slug/point-claims/*`.

**Facts that shaped it (not obvious from code):** LIFF SDK usage is thin — single `@line/liff` import in `src/stores/integration/index.ts` + `liff.state` query handling in HomeView, so a hand-written LIFF bootstrap in `main.tsx` is viable without waiting for `shell-liff`. `VITE_I18N_URL` is still "TBD" in `.env.development`, so `src/i18n/defaultKeywords.ts` (532 lines) is the real i18n source today — kit's compile-time catalog fits better than it first looks. member-api CORS is `AllowOrigins: "*"` (browser rejects with credentials → local dev needs same-origin Vite proxy, see [[reference_oc2plus_member_frontend]]).

**Why:** rewriting ~5,900 view LOC (2,867 of it scoped CSS) without an E2E net is unverifiable, and binding to a `0.0.1-spike.0` tarball whose api/auth surface is being redesigned in PAT-2660 means re-packing on every kit change.

**How to apply:** if asked to "start the React migration", begin with bucket 1 and the PAT-2660 comment, not with `create:local`. Related: [[reference_frontend_kit_state]], [[project_customer_app_program]], [[project_oc2plus_customer_app_auth_plan]].
