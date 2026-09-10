---
name: reference_oc2plus_e2e_playwright_repo
description: "OC2Plus e2e Playwright harness EXISTS and is mature — sellsuki/oc2plus/line-crm/oc2plus-line-crm-e2e-playwright (project 808), sits at the GROUP ROOT not under testing/; TS-009 Liff-Register = 45 TC live-verified against member.staging-th, 3 still skipped"
metadata:
  node_type: memory
  type: reference
---

**Verified 2026-09-10.** The customer-app E2E harness is **not** missing. It is
`git@gitlab.sellsuki.com:sellsuki/oc2plus/line-crm/oc2plus-line-crm-e2e-playwright.git`
(GitLab **project id 808**), created 2026-06-03, `main` active to 2026-09-08 with merged MRs.

⚠️ **Why two sessions concluded it didn't exist:** it sits at the **line-crm group root**, not
under `line-crm/testing/`. `git ls-remote .../testing/oc2plus-customer-app-e2e-playwright` and
`.../testing/oc2plus-line-crm-e2e-playwright` both return "not found" — both paths are wrong.
List the group instead: `glab api "groups/sellsuki%2Foc2plus%2Fline-crm/projects?include_subgroups=true&simple=true"`
**run from inside a submodule of that host** (from an unrelated cwd `glab` targets gitlab.com and
answers `Unauthenticated`, which reads like a permissions problem and is not one).

Structure: `playwright.config.ts`, `.gitlab-ci.yml`, `tests/` + `tests/fixtures/`,
`resources/pages/`, `variables/`, `utils/`, `payload/`, `plan/`, `docs/`, `AGENTS.md`, `.claude/`.
Allure reporting, `ENV=staging` switch, tag system (`@sanity_test`/`@regression_test`/`@smoke_test`/`@feature_*`).

Specs: TS-002 Campaign · TS-003 Point-Base-Rate · TS-008 Contact · **TS-009 Liff-Register** ·
**TS-010 Theme** · TS-011 Bola-Menu (+ real-account).

**Member/customer-app coverage that already exists:** `tests/TS-009-Liff-Register.spec.ts` = **45 TC**
(832 lines) written against the real `oc2plus-linecrm-frontend-member` `develop`, live-verified on
`member.staging-th.oc2.plus`, using fixture-based custom test fns (`testLiffRegister`,
`testLiffThemed`, `testLiffTenantIsolation`, …) rather than bare `test(` — so `grep -c "^test("`
returns **0** and looks like an empty suite. Only **3 TC still `.skip`**: TC-LIFF-REGISTER-12
(expired OTP), TC-26, **TC-33 (tenant isolation)** — the last because there is no seeding path for a
fixture-created company's slug. Most OTP TCs were un-skipped in July via route interception.
`TS-010-Theme.spec.ts` covers per-company theme.

**Not covered:** `/:slug/login`, `/:slug/point-claims`, `/:slug/point-claims/new`.

**How to apply:** OC-4370 is "adopt project 808 + fill gaps", not "build a harness"; OC-4491 is
"add 3 flows + unskip 3 TC" (size L → M). Recorded in comments 44640 (OC-4370) and 44641 (OC-4491).
Related: [[project_oc2plus_member_react_migration]] [[reference_oc2plus_member_frontend]]

## อัปเดต 2026-09-10 — clone เข้าโมโนรีโปแล้ว และ CI ไม่เคยเขียวเลย

**ตอนนี้ track แล้ว** ที่ `testing/oc2plus-line-crm-e2e-playwright` (monorepo commit `182bcb4`) · ก่อนหน้านี้ไม่อยู่ใน `.gitmodules` = มองไม่เห็นจากโมโนรีโป (อาการเดียวกับ ai-chat-e2e) · default branch **`main`** ไม่ใช่ develop · ยิงไปที่ `member.staging-th.oc2.plus`

### 🔴 CI พังเพราะ placeholder ที่ลืมแก้

`.gitlab-ci.yml:10` ตั้ง `AUTOMATE_REPOSITORY` → `sellsuki/oc2plus/oc2plus-e2e-test-playwright.git` ซึ่ง **404 Project Not Found** · ของจริงคือ `sellsuki/oc2plus/line-crm/oc2plus-line-crm-e2e-playwright` · มี `TODO[OC2PLUS]` กำกับบรรทัดบนอยู่แล้ว (scaffold มาจาก patona-e2e ไม่เคยแก้) → นี่คือเหตุของ `script_failure`

**pipeline ทั้งประวัติ = 3 ครั้ง: canceled / canceled / failed — ไม่เคยเขียว** · `projects/808/variables` = **0 ตัว** แต่ suite อ่าน env 18 ตัวรวม `POSTGRESQL_USER/PASSWORD`, `BOLA_DIAGNOSTIC_EMAIL/PASSWORD`, `BASEURL_MEMBER`, `KRATOS_URL` · **ไม่มี `.env` ในรีโป** → รันในเครื่องก็ไม่ได้ รันใน CI ก็ไม่ได้

**ผล: AC "E2E baseline เขียว" ของ OC-4495/4496/4497/4501–4505 ทุกใบพิงตาข่ายที่ไม่เคยมีอยู่จริง** — อาการเดียวกับ unit test ที่ `echo "1 + 1"` ก่อน OC-4492 · เขียนไว้ที่ OC-4370 comment 44662 + OC-4491 comment 44660/44661

### กับดักการนับ coverage

TS-009 ใช้ fixture wrapper เอง (`testLiffRegister`, `testLiffForcedScreen`, `testLiffRegisterMockable`, `testLiffForcedScreenMockable`, `testLiffThemeApiFail`, `testLiffThemed`, `testLiffTenantIsolation`) → **`grep -c "test("` ได้ 0 ทั้งไฟล์** · ของจริง 832 บรรทัด / **42 case invocation**

### coverage จริงของ customer app = register เท่านั้น

7 spec: TS-009-Liff-Register (customer app) · TS-002-Campaign · TS-003-Point-Base-Rate · TS-008-Contact · TS-010-Theme · TS-011-Bola-Menu(+real-account) = backoffice/BOLA

**ขาด 3 flow ไม่ใช่ 2**: member login (เบอร์+OTP) · point-claims list/detail · point-claims submit
- `point-claim`/`pointClaim`/`point_claim` → **0 ไฟล์ทั้งรีโป**
- ⚠️ `resources/pages/login.page.ts` = **login ของ staff/backoffice** (identifier+password+google+forgot) **ไม่ใช่ member** · `resources/pages/member.page.ts` = **หน้ารายชื่อสมาชิกในหลังบ้าน** ไม่ใช่ member app — ห้าม reuse ผิด
- 15 ไฟล์ยังมี `TODO[OC2PLUS]` ค้าง
