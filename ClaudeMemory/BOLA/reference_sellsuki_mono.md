---
name: Sellsuki Monorepo vs Standalone Repos
description: Critical mapping between /sellsuki/ (standalone clones) and /sellsuki_mono/ (monorepo with overmind) — same GitLab remotes, different local paths. Always edit in sellsuki_mono when running locally.
type: reference
---

## Two Local Copies — Same Remote

| Local Path | Purpose | Managed By |
|---|---|---|
| `/Users/nunt/sellsuki_mono/` | **Active dev environment** — overmind, Docker, Caddy, air hot-reload | `make dev` / overmind |
| `/Users/nunt/sellsuki/` | **Standalone clones** — same GitLab repos, used for isolated work | manual |

Both point to the same GitLab remotes. Changes made in one are NOT automatically reflected in the other.

**Rule: Always edit in `sellsuki_mono` when running the monorepo locally.** Edits in `/sellsuki/` won't take effect until synced.

## Key Services for Provider Management

| Service | Monorepo Path | Port | Caddy Domain |
|---|---|---|---|
| CCS Backend | `sellsuki_mono/backend/sellsuki-central-control-backend/` | 8092 | `ccs.sellsuki.local` |
| Management Backend | `sellsuki_mono/backend/management-backend/` | 8083 | `mgmt-api.sellsuki.local` |
| Provider Frontend | `sellsuki_mono/frontend/sellsuki-provider-management-frontend/` | 5179 | `provider.sellsuki.local` |
| Kratos UI | `sellsuki_mono/backend/kratos-ui-go/` | 4455 | `accounts.sellsuki.local` |

## Provider Frontend → mgmt-api (not CCS directly)

Frontend `.env.dev` must use `VITE_CCS_API_URL=https://mgmt-api.sellsuki.local` (management-backend on port 8083).

Management-backend routes: `/user/company`, `/user/company/:id`, `/user/profile`
CCS routes (not exposed via mgmt-api): `/v1/companies`, `/v1/company/:id`

## Known Vite 8 + Svelte Issues

- Svelte scoped CSS doesn't apply to `<main>`, `<ssk-*>` web components → use `:global()`
- `onMount` async doesn't fire reliably in svelte-routing context → use `{#await}` or immediate IIFE
- axios.client.ts `isDev` check must include `.sellsuki.local` domains for header injection
