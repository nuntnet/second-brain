---
name: reference_frontend_kit_state
description: "frontend-kit (sellsuki/share/library/frontend-kit, PAT-2587): all code on spike/walking-skeleton under Draft MR !1, main = README only, 0 packages on registry 856 — tarball-only; auth/Identity model is staff-app shaped; RN adapter is a type stub; scaffold verified to build 2026-09-08"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 0f82b824-5c8b-4490-a5fb-553341a58b63
  modified: 2026-09-08T16:49:40.840Z
---

**State verified 2026-09-08** (`git@gitlab.sellsuki.com:sellsuki/share/library/frontend-kit.git`, GitLab project id 856):
- `main` = single "Initial commit" (README only, 2026-08-05). All 39 commits live on `spike/walking-skeleton` under **MR !1 which is still Draft** (opened to get CI running, not to merge). Pipelines on the MR are green.
- **Nothing published.** `projects/856/packages` = `[]`; `publish_npm` needs a protected `vX.Y.Z` tag and none is configured. Only way to consume = `pnpm create:local <dir> --shell app|vue|svelte` (packs `file:./.kit-tarballs/*`) or `pnpm pack` per package. Version string `0.0.1-spike.0`.
- Toolchain: Node ≥22.18 (`.nvmrc` 22.23.2), pnpm 10.30.3. On this Mac `corepack enable` fails EACCES on /usr/local/bin; `corepack prepare pnpm@10.30.3 --activate` still works.
- **Boilerplate half works**: I scaffolded `--shell app` (React 19 + react-router 7 + TanStack Query 5 + nanostores + MSW), `pnpm build` ok (index 113 kB gz), `depcruise` 0 violations, `bootstrap.test.ts` 5/5. Note: with the seeded `.env` (`VITE_USE_MOCKS=true`) a plain `pnpm build` still emits the 94 kB gz msw chunk — unset it for a real deploy.
- Packages: `@sellsuki-org/foundation` (Result, AppError taxonomy mirroring Go `{error_code,error}`, `resolveErrorTreatment`), `@sellsuki-org/api` (keyed `configureApi(name)`, fetcher, orval mutator, `scopeHeadersFor` → `x-user-id`/`x-company-id`), `@sellsuki-org/core` (subpath-only: auth, permissions, workspace, flags, i18n, uploads, realtime — no root export, breaking refactor from 7 `core-*` pkgs already happened on the spike), adapters react/vue/svelte, `adapter-rn` = **interface-only stub, no code**, `shell-liff`/`shell-expo` = `ready:false`, no `design-tokens`, no telemetry/notifications/analytics yet.
- **Auth model is staff-app shaped**: `createAppRuntime` requires `Identity {userId, companyId}` at boot; `AuthPort.bootstrap()` expects session+workspace+flags+permissions in one call (Kratos/Keto pattern). A cookie-only customer app (OC2Plus member: `oc2plus_crm_session` + company slug in URL, `/session/whoami` returns only member_id/integration_id/expire_at) needs its own AuthPort adapter — nobody has written one. Kit itself warns the identity headers are CLIENT-ASSERTED (cf. AI-136).
- `core/i18n` = compile-time typed TH/EN catalog + Intl wrappers (พ.ศ., THB, Asia/Bangkok); no remote-keyword port for i18n-management-backend.
- DS in React proven in `apps/skeleton` but JSX typings hand-written for 3 components only, and the side-effect DS import costs 160→583 kB gz (kit's own measurement, blocks its FK-8.4 budget).
- README claims "library half proven by 3 production repos" — **unverifiable from the monorepo**: grep `@sellsuki-org/foundation` across every frontend package.json = 0 hits; MR !1 description names no repo either.
- Jira: epic PAT-2587 (To Do; design of record = claude artifact + `docs/ai-chat-assistant-platform-plan.md` §5.5.1/§5.6.11). Cards: PAT-2591 spike Done · PAT-2659 Phase1 repo/CI In Progress · PAT-2660 Phase2 api/auth/permissions In Progress · PAT-2676 Phase3 core ports To Do (7 subtasks 2677-2683). First consumer = AI-118 (ai-chat-admin) still To Do. Epic rule already decided: React for all new apps incl. LIFF + Expo; legacy migration = route-level strangler behind one domain; RN mapping of sellsuki-components is out of scope.

See [[project_oc2plus_member_react_migration]] for how this applies to the OC2Plus member frontend.
