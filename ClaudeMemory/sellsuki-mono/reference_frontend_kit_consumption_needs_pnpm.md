---
name: reference_frontend_kit_consumption_needs_pnpm
description: "Consuming frontend-kit via the vendor tarball REQUIRES pnpm — create-local.mjs writes a pnpm.overrides block npm ignores; SRE's frontend CI parameterizes test scripts but hardcodes `npm install` in the build job, and only configures the @sellsuki scope while the kit is @sellsuki-org"
metadata:
  node_type: memory
  type: reference
---

**Verified 2026-09-10** against `frontend-kit` `spike/walking-skeleton` (project 856) and
`sellsuki/sre/deployment/pipeline-deployment` `main`.

## The tarball path is pnpm-only

`scripts/create-local.mjs:113-138`: it packs with **`pnpm pack`, not `npm pack`**, because npm leaves
`workspace:*` verbatim in the packed manifest while pnpm rewrites it to `0.0.1-spike.0`. That version
exists on **no registry**, so the script then writes into the scaffolded app's manifest:

```js
manifest.pnpm = { ...manifest.pnpm, overrides: { ...tarballs } }   // line 138
```

`pnpm.overrides` redirects every request for `@sellsuki-org/*` — the app's own **and each tarball's
transitive ones** — at the local files. **npm does not read that field at all**, so `npm install` on a
scaffolded app resolves kit→kit deps from the registry and 404s. npm can replicate it with a top-level
`overrides` map, but that is hand-written and must be regenerated on every re-pack.

⚠️ So "npm can consume the kit either way" is **false**. Registry path → npm is fine (real versions,
no overrides). Tarball path → pnpm, or ongoing manual npm `overrides` maintenance. I asserted the
wrong version of this in OC-4492 comment 44638 and corrected it in 44642.

## The kit repo is a pnpm workspace (dev side, not consumer side)

Root `package.json`: `"packageManager": "pnpm@10.30.3"`, `"engines": {"node": ">=22.18.0"}`, every
script is `pnpm -r …`, turbo + changesets. Layers under `packages/`: `domain`, `application`,
`boundary`, `adapters`, `tooling`. That toolchain is a requirement for *developing* the kit — do not
read it as a requirement on repos that merely consume it.

## SRE frontend CI: tests are parameterized, the build is not

- `pipelines/gitlab-ci-pipeline.generic-frontend-vue-npm-th.yml` is **18 lines** and only sets
  defaults: `UNIT_TEST_BEFORE_SCRIPT: "npm install"`, `UNIT_TEST_SCRIPT: "npm run test:unit"`
  (+ the E2E/INTEGRATION pair), then includes `generic-frontend-npm-th.yml`. **The template already
  runs real unit tests** — a repo showing `echo "1 + 1"` is overriding it locally. Turning tests on =
  deleting the override, not writing a job.
- Because those are variables, **pnpm in the test jobs needs no new SRE template**.
- `templates/build.template.yml:64-76` (`.build_frontend_npm`) **hardcodes** in `script:`
  `npm install` + `npm run build:${APP_ENV}`; only `before_script` is a variable
  (`FRONTEND_BEFORE_SCRIPT`). So **building for deploy with pnpm genuinely needs a new SRE template.**
- Only four frontend templates exist: `generic-frontend-{npm,npm-th,vue-npm,vue-npm-th}` — no pnpm/yarn/bun.
- `DOCKER_IMAGE` is set per-env in each repo's own `.gitlab-ci.yml`, so a Node bump (20→22) is a
  repo-local one-liner.

## Unrecorded gap: wrong npm scope in CI

`build.template.yml:73-74` configures the registry and `_authToken` for scope **`@sellsuki`** only.
The kit publishes under **`@sellsuki-org`**. CI cannot install kit packages until a config line for
that scope is added — true for npm and pnpm, and true for the tarball path too (transitive deps still
resolve from the registry). Filed on OC-4499 (comment 44643) and OC-4492.

**How to apply:** the package-manager decision belongs to the first card that actually installs the
kit (OC-4499), never to a toolchain/CI card in the prep lane — M1–M8 never install it.
Related: [[reference_frontend_kit_state]] [[project_oc2plus_member_react_migration]]

## node:22 on the internal mirror is UNVERIFIED (2026-09-10)

Every frontend `.gitlab-ci.yml` in this monorepo pins
`registry.fountain.sellsuki.com/dockerhub/library/node:20` — ai-chat-admin-frontend,
bola-frontend, sellercenter, company/provider/invitation, oc2plus backoffice and member.
**Nothing here has ever used node:22.** The registry needs auth (anonymous
`/v2/.../tags/list` and a manifest HEAD both return 401), so tag availability cannot be
checked from the CLI. If a pipeline that bumps to node:22 dies on image pull, that is the
cause, and the fallback is node:20 — Vite 5 + vitest 1 run fine on it; only frontend-kit
(Node >= 22.18) actually needs 22, and that lands with OC-4499.

