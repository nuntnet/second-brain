---
name: reference-ccs3-frontend-facts
description: "CCS3 = sellsuki-company-management-frontend at admin.sellsuki.com; members+invite UI is /users (no /companies/:id/members), company comes from session selectedCompanyId not the URL; local run needs bun install for the DS package"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T13:48:45.453Z
---

`frontend/sellsuki-company-management-frontend` — the app everyone calls **CCS3**. Facts that are easy to guess wrong (I did, in the BOLA-285 "Manage in CCS" deep-link — it pointed at a URL that never existed):

- **Deployed hosts** (from its `.gitlab-ci.yml` `APP_URL`): prod `admin.sellsuki.com`, staging `admin.staging.sellsuki.com`, dev `admin.dev.sellsuki.com`. **There is no `ccs.sellsuki.com` UI** — `ccs.sellsuki.local` in the local Caddyfile is the CCS *backend* (:8092).
- **Members + invite management lives at `/users`** (`src/App.svelte` route table, svelte-routing). There is no `/companies/:id/members` route. Other routes: `/company`, `/company/edit`, `/group[/new|/view/:roleId|/edit/:roleId]`, `/pay/...`, `/location`.
- **Company is resolved from the session**, `config.data.user.selectedCompanyId` (`src/stores/company/current-company.ts`) — never from the URL. So a deep link from another product **cannot preselect a company**: an ops user with a different company selected lands on the wrong company's member list. Fixing that needs CCS3 to accept a company query param (open card if it matters).
- **Local run**: `company.sellsuki.local` → :5177, started by `web-company` in `Procfile`/`Procfile.frontend`. Its committed `bun.lock` was **missing the declared dep `@sellsuki-org/sellsuki-components`** (observed 2026-08-11 at develop `a171ebb`) — vite boots and the port listens, but the app dies on "dependencies imported but could not be resolved" and the page is broken. Fix: `bun install` in that repo (this rewrites `bun.lock`, +~99 lines — a real lockfile repair worth committing). `~/.npmrc` maps only the `@sellsuki` scope, yet the `@sellsuki-org` package resolves fine through the host-level token.

- **Local auth = dev-bypass header, and its UUID is machine-specific.** `.env.dev` carries `VITE_ADDITIONS_HEADERS={"X-User-Id": "<uuid>", "X-User-Kind": "sellsuki.user"}`. The **committed** UUID (`d46da77b-…`) does not exist in a locally-seeded Kratos, and a committed UUID can never be portable — Kratos mints random ids per setup. Symptom when it's wrong: `GET /v1/user/profile` returns **HTTP 500 wrapping `404 Not Found`** (CCS asked Kratos, Kratos 404'd) → `loadProfile` gets undefined → `navigateToLogin()` → `accounts.sellsuki.local/login` has no session → bounces back → **infinite redirect loop** with `?error=&error=&return_to=…&theme=` accumulating.
  - Fix: `curl -s -o f "http://localhost:4434/admin/identities?per_page=50"` (write to a FILE — the rtk hook summarizes curl JSON into a schema), pick an id whose `/v1/user/profile` returns 200 **and** whose `/v1/user/company?provider_code=sellsuki` returns ≥1 company (the store falls back to `cs[cs.length-1]`, so zero companies is its own dead end), put it in `.env.dev`, restart `web-company`.
  - Verify without a browser: `curl -sk https://company.sellsuki.local/src/lib/axios-client-v2.ts | grep <uuid>` — vite inlines env into served modules, so the new id appearing there proves the restart took.

See [[reference-local-bola-own-overmind-socket]] for the per-stack overmind socket pattern used to start it without disturbing other sessions.

## Branch -> environment (verified 2026-09-10 from the deployments API)

- **`develop` -> development** (deploys reliably; `develop` 30e6de8a succeeded 2026-09-10 10:26)
- **`main` -> staging** — last *successful* `deploy_staging` was 2026-09-04 at `main` a4185a1d.
  A later run at `main` 12d55d68 (2026-09-07) came back **`skipped`**, so "there is a
  staging deployment" and "the last pipeline deployed" are different questions — filter
  the deployments API by `status=success` before concluding anything.
- **production <- tags** (last: `v1.8.0`, 2026-07-14)

**Correction:** an earlier note in this workspace claimed CCS3 frontend has *no* staging
environment. That is wrong — environment ids are staging 396, production 397,
development 400. The confusion came from reading a `skipped` deployment as an absent one.

`develop` runs far ahead of `main` (119 commits ahead, 9 behind on 2026-09-10), so a
feature merged to `develop` is on **development only** and invisible on staging until
someone merges `develop` -> `main`. The BOLA Workspaces menu hit exactly this: present on
`develop` (`src/pages/BolaWorkspace/`, 3 files + the sidebar entry), **absent from `main`**.

Sidebar menus here are gated by env flag only, no permission check —
`VITE_DISABLE_<FEATURE>_FEATURE` read through a local `toBoolean` in
`src/components/Layout.svelte:53` that correctly treats the string `"false"` as false
(`value.toLowerCase() === 'true'`), so a quoted `"false"` in `.env.*` does NOT hide the menu.

### Staging deploy is a manual gate, and `prepare_staging_frontend` gets stuck

Merging to `main` does **not** put CCS3 on staging by itself. The pipeline runs
`prepare_staging_frontend` -> tests -> `build_code_staging` automatically, then
**`deploy_staging` sits at `manual`** waiting for a click (job play via
`POST /projects/:id/jobs/<id>/play`).

`prepare_staging_frontend` is also the flaky one: the 2026-09-07 run on `main`
12d55d68 failed with `stuck_or_timeout_failure` (no runner claimed it), which cascaded
every later job to `skipped` — so staging silently stayed on `main` a4185a1d from
2026-09-04 for three days with nobody noticing. It succeeded on the retry
(pipeline 58139, 2026-09-10) with no code change, so treat it as transient — but
check `failure_reason` before blaming code. See [[reference_dead_staging_runner_tag]]
and [[reference_stuck_ci_job_holds_resource_group]] (cancel, do not retry, when a job
holds a resource_group).

CI installs with **`npm ci`** and runs `npm run test:ci` (= `vitest run && npm run check`),
so `package-lock.json` is what CI needs while `bun.lock` serves local dev only. A commit
that adds a dep on `main` therefore updates package-lock and leaves bun.lock stale —
after merging main into develop, `bun install --frozen-lockfile` fails while CI stays
green. Regenerate bun.lock as part of any such sync (done in !505 for `jsdom`).
