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


## 2026-09-11 (evening) — it was live on `develop` and worse than first reported

PO hit it in the browser. On `develop`, `PUBLIC_REACT_ROUTE_NAMES` held only
`react-probe`, so **every** react-owned route sat behind the guard — including
`login`, which is AuthGuard's own redirect target. Measured on the local dev
server: a real page load of `/{slug}/history` settled at `/{slug}/login` with
`document.body` empty, 0 child nodes, no `data-testid`. `/{slug}/home` the same.
The guard rejected its own redirect and rendered `null`, so the whole app was a
dead end for anyone not already signed in.

`publicRoutes.spec.ts` **asserted the bug** (`isPublicRouteName('login') === false`),
and `LoginPage.spec.tsx` renders the page directly, bypassing `AppShell` — which is
why 490 unit tests and type-check stayed green over a completely unusable app.

Fixed in **MR !68** (`fix/oc-4501-login-blank-for-anonymous`), a one-line change plus
a spec that locks the general rule: **a route that RESOLVES the "no session" state can
never be gated on having one.** Verified after: same navigation renders `login.page`,
`login.phone.input.phone`, `login.phone.button.submit`. MR !65 (OC-4502) carries the
same fix plus `register-by-slug`, but !68 is small enough to merge first.

**Diagnostic that works:** unit tests cannot see this. Load the page for real and read
`document.body.innerText.length` / child count — a guard-blanked page is 0/0 while the
URL looks correct.
