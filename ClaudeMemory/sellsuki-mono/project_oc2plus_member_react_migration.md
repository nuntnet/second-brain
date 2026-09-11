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
1. Do now, kit-agnostic: E2E harness on web surface (OC-4370 — ~~repo does not exist~~ **WRONG — corrected 2026-09-10: harness exists as `oc2plus-line-crm-e2e-playwright` project 808 at the line-crm GROUP ROOT, with 45 member-register TC already live on staging; see [[reference_oc2plus_e2e_playwright_repo]]**; member CI unit/e2e are `echo "1 + 1"` stubs; only 9 vitest spec files), pull logic out of the 5 views into pure TS `domain/dto/usecases` returning `Result`, 8-line `call()` boundary, upgrade DS `sellsuki-components@0.7.71` (unscoped) → `@sellsuki-org/sellsuki-components@0.27` (still exports `applyTheme`, `IdbI18nStore`, `ssk-theme-provider/text/image/icon`), prep CI for Node 22 + pnpm (current template `generic-frontend-vue-npm-th`, node:20).
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

## Sprint plan เคาะ 2026-09-10 (PO)
**Decision 1 — Sprint 129 = prep lane, ไม่รอ kit.** Gate ยังไม่ผ่าน (ตรวจสด 2026-09-10: `projects/856/packages` = `[]`, MR !1 ยัง Draft, PAT-2660 In Progress updated 09-08 = ไม่ขยับจาก 09-08). **OC-4346 ถอด sprint ออกแล้ว** (`customfield_10020: null`, verified) เพราะ blocked-by OC-4499+OC-4500 ซึ่งรอ kit — Sprint 129 เริ่ม 2026-09-20 จะเปิดมาแล้วติดล็อกวันแรก
Sprint ที่เซ็ตจริง (verified ด้วย JQL): **129 (id 1365)** = OC-4490 M0, OC-4491 M1, OC-4492 M2 (+ ของเดิม OC-4347, OC-4348, OC-4370) · **130 (id 1366)** = OC-4493 M3, OC-4494 M4 · **131 (id 1991 — ไม่ใช่ 1367!)** = OC-4495 M5, OC-4496 M6 · OC-4497 M7 ยังไม่ใส่ (จะเป็น 132)
**Decision 2 — OC-4523 (verified LINE login) ทำบน Vue ก่อน แล้วย้ายพร้อม OC-4501.** BE (member-api `POST /company/{slug}/auth/line` + shell ส่ง id_token) ไม่ผูกสแตก FE เลย; FE แตะ `LoginView.vue` (694 LOC). ผลผูกพัน: OC-4501 ต้องครอบ LINE-login path ไม่ใช่แค่ phone+OTP (link Relates OC-4523↔OC-4501 สร้างแล้ว) และ logic ใหม่ต้องเขียนเป็น pure TS usecase คืน Result ตาม pattern M4 ตั้งแต่แรก

## ข้อเท็จจริงที่แก้จากรอบก่อน (ตรวจ 2026-09-10 ที่ develop 0314634)
- **ไม่มี dual lockfile ในรีโป** — `git ls-files` เจอแค่ `package-lock.json`; `bun.lock` เป็น untracked ในเครื่อง. ดังนั้น blocker ของ OC-4492 คือ "SRE ไม่มี pnpm template" เท่านั้น และ**ตัดออกได้**: pnpm 10.30 เป็นข้อกำหนดของการ *พัฒนา* kit ไม่ใช่การ *consume* (npm ติดตั้งทั้ง registry version และ tarball ได้) → เสนอ scope M2 = npm + Node 22 + เปิด unit test จริง แล้ว DoR ผ่าน (comment 44638)
- **OC-4490 Overview มี premise ผิด**: เขียนว่า "ไม่มี endpoint อ่าน identity ตอน bootstrap" — ผิด, member-api มี `GET /session/whoami`; ที่ grep 0 hit คือฝั่ง FE. ช่องว่างจริง = whoami ไม่คืน `company_id` vs kit บังคับ `Identity{userId, companyId}` (comment 44639)
- views ตอนนี้ 9 ไฟล์ **5,327 LOC** (PointClaimNewView 1452, MembershipView 1152, PointClaimDetail 812, PointClaimList 795, LoginView 694, HomeView 197)
- OC-4498 (M8) **merged เข้า develop แล้ว** (!37, commits 5226c84+9c9626e) และ !38 develop→main ยังเปิด — status board = code review
- `package.json` name ยังเป็น `vue-projectoc2plus-linecrm-frontend-backoffice`, CI = template `generic-frontend-vue-npm-th` + `node:20`, UNIT/E2E/INTEGRATION script ยัง `echo "1 + 1"`


## Sprint ladder ที่จัดใหม่ 2026-09-10 (ของเดิมไม่เคยลงจริง)

ก่อนแก้ **15 จาก 17 การ์ดกองอยู่ Sprint 129 พร้อมกันหมด** รวม OC-4506 (ปลด Vue) · มีแค่ 4495/4496 อยู่ 131 · Sprint 130 ว่าง → บอร์ดอ่านว่าย้ายทั้งแอปจบใน sprint เดียว

ladder ที่ลงจริงแล้ว (ยืนยันด้วย JQL):

* **129 (1365)** — 4490 ประสาน kit · 4491 E2E baseline · 4492 CI gate (**Done แล้ว**) · 4493 token · 4494 usecase login → ไม่พึ่ง kit เลย
* **130 (1366)** — 4495 · 4496 · 4497 usecase ที่เหลือ
* **131 (1991)** — 4499 React boot · 4500 strangler → ใบแรกที่ install kit จริง ต้องรอ 4490
* **ไม่มี sprint** — 4501–4506 (หน้าที่ต้องย้าย) รอ 4499/4500 + 4 ใบแรกยังมี 🔴 design ค้าง

OC-4498 คงไว้ 129 (การ์ด product ที่พลอยติด label)

⚠️ ในตัว description ของแต่ละการ์ดยังมีบรรทัด "Size / Sprint ที่เสนอ" ค่าเก่าคนละค่า (129/131/132/133/134/135) — **เลิกใช้ ให้ยึด sprint field** · ไม่แก้ใน description เพราะ rewrite ยาวผ่าน API ทำภาษาไทยเพี้ยน (ดู [[reference_jira_editissue_adf_breakage]]) · ladder ฉบับจริงอยู่ที่ OC-4344 comment 44656

## ชุดแรกลง develop จริงแล้ว 2026-09-11 (ทั้ง 4 ใบ + develop เขียวหลังรวม)

| MR | การ์ด | merge commit | develop pipeline หลัง merge |
|---|---|---|---|
| !42 | OC-4494 `src/usecases/result.ts` + auth usecases | 80b3a17 | 58278 ✅ |
| !43 | OC-4495 membership usecases | eeefd31 | 58290 ✅ |
| !44 | OC-4493 slice 1 design token | ad1283c | 58291 ✅ |
| !46 | OC-4493 slice 2 ปลด `ssk-*` | 62d288d | 58301 ✅ |

**พิสูจน์ด้วยของจริงบน develop ไม่ใช่ badge:** `git grep ssk- origin/develop -- '*.vue'` = **0 ไฟล์**

**บทเรียนที่ต้องทำซ้ำทุกครั้ง:** ยิง pipeline ใหม่ก่อน merge ทุกใบ (58279/58280/58293) เพราะ develop ขยับไปแล้วจาก OC-4462 + ใบก่อนหน้าในชุดเดียวกัน — badge เดิมของ MR พิสูจน์คู่ combination ที่ไม่มีอยู่จริงแล้ว · และ GitLab **ไม่ retarget stacked MR ให้อัตโนมัติ** หลัง parent merge ต้อง `PUT projects/529/merge_requests/{iid}` `target_branch=develop` เอง

**ผลข้างเคียงที่ user มองเห็น (OC-4493):** `src/theme/defaultTheme.ts` `primary` เปลี่ยน `#1A73E8` → `#32a9ff` — บริษัทที่ไม่ได้ตั้ง theme สีเปลี่ยนจริง ไม่ใช่ no-op (เคยจดผิดว่าเท่ากันอยู่แล้ว ดู [[project_oc2plus_member_app_design_v2]])

**OC-4532 (CI boundary gate) merged `f095979` 2026-09-11** — หลักฐาน 3 รอบบน MR !47: เขียว 58300 (`b492520`) → แดง 58306 (`111e46b` จงใจ `import { ref } from 'vue'` ลง `src/usecases/auth/login.ts`) → เขียว 58307 (`a7e46d2` revert, ไม่ amend ไม่ force push). `.gitlab-ci.yml:27` บน develop = `UNIT_TEST_SCRIPT: './ci/unit-test-run.sh'`, mode `100755`.
**สิ่งที่รอบแดงพิสูจน์เกินกว่า "แดงได้":** unit test ยังผ่าน 233/233 แต่ job แดง และ log ปิดท้ายด้วย `=== [ci] summary: unit-test=0 boundaries=1 ===` — ผลของการ**จงใจไม่ใส่ `set -e`** ในสคริปต์ ถ้าใส่ log จะจบแค่ "job failed" แยกไม่ออกว่าเทสหรือ boundary. `depcruise` บน 1,257 modules = ~1.7s; เทียบ duration job ตรง ๆ ไม่ได้เพราะ `unit_test_merge_request` แกว่ง 86–135s ตาม branch/`npm ci` อยู่แล้ว

## 2026-09-11 — state after other sessions ran ahead
- **PO decision (OC-4490 comment 44680, OC-4499 comment 44679): React runtime is built WITHOUT frontend-kit** (plain React 19 + Vite in the same repo, `src/react/`, `src/main.tsx`, `src/routeTable.ts`). Kit adoption deferred to registry-published version only; no `.kit-tarballs` in repo. OC-4490 stays as a parallel coordination task, not a blocker.
- Merged to `develop` by 2026-09-11: OC-4492 CI tests, OC-4493 design tokens, OC-4494/4495/4496/4497 usecases (`src/usecases/{auth,membership,pointClaim}`), OC-4498, OC-4499 React probe + `SessionService.whoAmI` + `useSession`, OC-4500 strangler routing (`routeTable.ts`), OC-4346 app shell (`src/react/shell/AuthGuard`), OC-4501 login React port, OC-4535 design v3 tokens. **Jira status lags code**: OC-4346/4500/4501 still "To Do" though merged. Epic-2 page branches `feat/oc-4350..4361-page` exist.
- **Requirement to Patona posted: PAT-2660 comment 44698** (6 items: optional companyId / anonymous boot, minimal AuthPort w/o workspace-permissions-flags + optional displayName, opt-in scope headers, taxonomy case+404 handling, registry publish (tarball is pnpm-only; SRE template only `@sellsuki` scope), reserve shell-liff). OC-4490 progress comment 44700-ish posted same day. Deadline for answers: before OC Sprint 133.
- dev-th quirk found by another session: `GET /session/whoami` without cookie returns 200 with zero-value body instead of 401 → FE treats empty member_id as anonymous.
- E2E harness truth: project 808 `oc2plus-line-crm-e2e-playwright` at line-crm group ROOT (now submodule `testing/oc2plus-line-crm-e2e-playwright`), 42 register TCs but CI never green (placeholder repo URL, 0 CI vars). OC-4491 = add login/point-claims flows.

## 2026-09-11 (evening) — OC-4491/4370/4502 delivered, two shipped bugs found
- **E2E baseline now real.** project 808 MR !11 (`feat/oc-4491-point-claims-flows`) adds TS-013 (list/detail) + TS-014 (submit) + point-claims fixtures/page-objects/resource, plus OC-4370 harness certification (README/AGENTS, `utils/.env.staging.example` with all 21 env vars, `docs/liff-manual-smoke-checklist.md`, `docs/otp-standard.md`). **I re-ran it myself: TS-012+013+014 = 45/45 pass in 15s** against `member.staging-th.oc2.plus` with only `BASEURL_MEMBER` set. TS-009+TS-012 earlier = 56 pass / 1 fail (TC-29 needs Kratos). Route interception throughout, no secrets, no Thaibulk.
- **OC-4502 delivered**: member FE MR !65 (`feat/oc-4502-register-react`), register ported to React, 485 unit tests, TS-009 41 pass/3 skip/1 env-fail against a local `vite preview`, rollback to Vue exercised.
- 🔴 **OC-4501 shipped a blank login page** — see [[reference_react_route_owner_flip_needs_public_allowlist]]. Fixed inside MR !65. Not on `main` yet (main has only the OC-4499 probe; 10 commits pending on develop). **Do not merge develop→main until !65 lands.** Recorded at OC-4501 comment 44702.
- 🔴 **OC-4537 opened** (Bug, parent OC-4344): (1) `PointClaimDetailView.vue:66-69` redirects a 401 to `register-by-slug` while `PointClaimListView.vue:95-98` correctly redirects to `login` — an existing member with an expired session is sent to signup; (2) `services/pointClaim/index.ts:74` checks `status === 400` before the `error_code` branches at `:83`/`:86`, so `VALIDATION_FAILED` and `PHOTO_INVALID` are dead code and surface as `invalid_pagination` — **and the comment at `:80-82` claims the opposite of what the code does**. Same bug class the repo already documented in `services/auth/index.ts` (404-slug read as member_not_found) and that frontend-kit's README warns about.
- **E2E binds to CSS classes, not only testids** (`h3.title`, `.control--error`, `.name-row` siblings, `getComputedStyle` on `--c-primary`) — a pure inline-style port compiles, passes unit tests, and reds the whole suite. Matters for OC-4503/4504/4505.
