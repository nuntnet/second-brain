---
name: reference_react_route_owner_flip_needs_public_allowlist
description: "OC2Plus member FE strangler: flipping a route to owner:'react' in routeTable.ts silently puts it behind AuthGuard — a public entry (login/register) then renders blank with a same-URL pushState loop, and unit tests never catch it"
metadata:
  node_type: memory
  type: reference
---

In `frontend/oc2plus-linecrm-frontend-member`, the React strangler has **two independent
switches that must be flipped together**, and nothing enforces the pair:

1. `src/routeTable.ts` — `owner: 'vue' | 'react'`
2. `src/react/shell/publicRoutes.ts` — `PUBLIC_REACT_ROUTE_NAMES`

`AppShell.tsx`: a react-owned route with a `REACT_PAGES` entry that is **not** in the public
set is wrapped in `ShellPage` → `ShellPageImpl` → `<AuthGuard>`. `AuthGuard` on an anonymous
session does `window.history.pushState(…, '/{slug}/login')` and returns `null`.

**So flipping a PUBLIC entry to react without adding it to the allow-list makes that page
render blank forever**, pushing the same URL it is already on. Shipped exactly this way in
OC-4501 (`e8dbb2f`, on `develop` 2026-09-11): `/:slug/login` blank for every anonymous
visitor — i.e. everyone who needs it. Caught during OC-4502, fixed in
`feat/oc-4502-register-react`; never reached `main` (main had only the OC-4499 probe).

**Why the suite stayed green:** `LoginPage.spec.tsx` renders the page component directly,
bypassing `AppShell`, so `publicRoutes` and `AuthGuard` are never exercised. Type-check and
unit tests both pass. The guard is only reachable through the shell.

**How to apply:** when porting any route to React, change `routeTable.ts` and
`publicRoutes.ts` in the same commit, and keep the test that asserts **every** `owner:'react'`
route has a deliberate public/protected decision. Public entries today: `react-probe`,
`login`, `register-by-slug`. Related: [[project_oc2plus_member_react_migration]]
