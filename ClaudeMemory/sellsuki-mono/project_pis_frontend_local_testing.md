---
name: project-pis-frontend-local-testing
description: pis-frontend defaults to dev-th STAGING API; how to wire it to local pis-api for testing
metadata: 
  node_type: memory
  type: project
  originSessionId: 9e057444-0352-41d3-9463-fb0dd304ba9a
---

`frontend/pis-frontend` `.env.development` sets `VITE_APP_BASE_API_URL=https://api.dev-th.sellsuki.com/pis`
— so the local dev server (overmind `web-pis`, port 5185, mode `development`) **talks to STAGING by
default, NOT local `pis-api`**. The permission/session-expired error on `pis-app.sellsuki.local`
comes from staging, not local.

**Why:** PIS is an embedded-in-CCS3 micro-frontend; locally there's no Kratos→identity gateway, and
the env points at staging. To test local PIS code (e.g. [[project_merchant_portal]] / MS-623) through
the UI you must repoint + fake identity.

**How to apply (test local pis-api through the PIS UI) — use SAME-ORIGIN to avoid CORS:**
1. `frontend/pis-frontend/.env.development.local` (gitignored via `*.local`):
   `VITE_APP_BASE_API_URL=https://pis-app.sellsuki.local` (SAME origin as the SPA → zero CORS).
   (Cross-origin `https://pis.sellsuki.local` also works server-side but the browser hits cached-
   preflight / transition CORS pain — same-origin is robust.)
2. `Caddyfile` `pis-app.sellsuki.local` block: route API paths to pis-api, rest to Vite, inject identity:
   `@api path /v2/* /permissions /session` → `reverse_proxy localhost:8081` with
   `header_up X-User-Id d46da77b-5f10-479a-9006-f91939a0b85f` + `X-User-Kind sellsuki.user`
   (DEV_USER_ID from `scripts/seed-dev.sh`; pis-api `CheckIdentity` rejects empty `X-User-Id`);
   `handle {}` → `reverse_proxy localhost:5185` (Vite). pis-api serves API at root (`/permissions`,
   `/v2/*`); the staging gateway is what adds the `/pis/*` prefix.
   Then `docker restart sellsuki_mono-caddy-1` (macOS bind-mount can serve a stale view → restart,
   not just `caddy reload`). Hard-refresh the browser after `overmind restart web-pis` (new bundle).
3. Grant PIS perms in LOCAL role-permission (seed-dev only grants `patona.customer.*`, never
   `sellsuki.productsystem.*`): gRPC `localhost:9998` `RoleAndPermissionService/CreateRole`
   (perms `sellsuki.productsystem.{view,create,update,delete}`, owner `{sukispace, sellsuki.company}`)
   then `AssignRole` (tenant `{sellsuki.company, sukispace}`, user = DEV_USER_ID). Catalog already has
   these 4 perms in `permission_lists` (group `product_management`).
4. `overmind restart web-pis`; open `https://pis-app.sellsuki.local/?ref_id=sellsuki.company:sukispace`
   (the `?ref_id=` query is REQUIRED — `routeGuard.ts` reads it into the iframe store; without it →
   session-expired). pis DB is named `products` (Postgres). MS-623 weight/dimension fields appear on a
   Physical product only after picking single/multi structure.
