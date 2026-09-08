---
name: project_oc2plus_member_react_migration
description: "OC2Plus member frontend → React + frontend-kit strangler: 17 cards OC-4490..OC-4506 written 2026-09-09 under epic OC-4344 (label member-fe-react), decisions = single build w/ Vue fallback, CSS modules first, vendor tarball w/ deadline; /session/whoami EXISTS in member-api (lacks company_id)"
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

## Cards written 2026-09-09 (user confirmed all 4 decisions)
Parent **OC-4344**, label `member-fe-react` + `customer-app-program`, no sprint field set (proposed sprints in text): OC-4490 M0 kit coordination (Task) · OC-4491 M1 E2E golden-path baseline · OC-4492 M2 bun→pnpm + Node 22 + real CI tests · OC-4493 M3 DS→0.27 scoped · OC-4494 M4 auth usecases · OC-4495 M5 membership usecases · OC-4496 M6 claim read · OC-4497 M7 claim write · OC-4498 M8 Bug 401/403/404 swallowed · OC-4499 M9 React runtime boot · OC-4500 M10 strangler routing · OC-4346 (existing) = M11 shell · OC-4501 login · OC-4502 register · OC-4503 list+detail · OC-4504 new · OC-4505 home/error/LIFF · OC-4506 remove Vue. 39 issue links. Comments on OC-4370/4346/4345/4465.
**Decisions:** strangler = single build, React entry, Vue mounted as fallback for unmigrated paths (NOT ingress split) · CSS = lift scoped CSS to CSS modules first, restyle with DS tokens later · kit = vendor tarball `0.0.1-spike.0` with deadline "before OC-4501 (Sprint 133)" · label `member-fe-react`.
**DoR state:** 4/17 ready (4490, 4494, 4496, 4498); blocked 4492 (SRE has no pnpm CI template, repo has both bun.lock + package-lock.json), 4499, 4505.
**Correction made:** po-lead claimed "no whoami endpoint" — wrong (grepped FE only). member-api HAS `GET /session/whoami` (`route/session_v1.go:10`, cookie-auth, returns member_id/integration_id/issued_at/expire_at, no company_id). Fixed in OC-4345 comment 44571 and OC-4499 comment 44573. Remaining real gap = company_id absent vs kit `Identity{userId, companyId}`.
**Facts corrected by po-lead vs my brief:** repo uses **bun** (`bun.lock`), views = 9 files 5,181 LOC, services don't import Vue (extraction target is the Pinia stores), theme loaded in 5 views not App.vue and `/` has no theme, router has no navigation guard, OC-4346 already has sprint field 129 which conflicts with its new blocked-by 4499/4500.
