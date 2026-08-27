---
name: reference-browser-surfaces-this-workspace
description: the in-app Browser pane DOES reach localhost here; what actually blocks verification is per-app auth and missing deps
metadata:
  type: reference
---

**Corrected 2026-08-27.** The earlier note that the Browser pane times out on
this workspace is wrong. `preview_start` with `http://localhost:<port>` works,
loads the page, and `read_network_requests` / `read_console_messages` return
real data. Verify in the Browser pane before assuming it cannot be done.

What actually blocks browser verification here, per app:

- **Backoffice SPAs (e.g. 5176)** — load, then redirect to Kratos at
  `accounts.sellsuki.local`, which refuses a `return_to` of
  `http://localhost:5176/` ("Requested return_to URL is not allowed"). Reaching
  a logged-in view needs the user to sign in; see
  [[feedback-verify-as-the-user-sees-it]].
- **`frontend/oc2plus-linecrm-frontend-member` (5183)** — serves HTML but
  `src/main.ts` 500s, so nothing renders. Its `node_modules` is a **symlink to
  the backoffice's**, and three of its own deps are simply absent there:
  `sellsuki-components`, `@sentry/vue`, `@line/liff`. Same root cause as its
  `npm run build-only` failing at baseline.

⚠️ **Do not run `npm install` in that repo.** It needs auth for the private
`sellsuki-components` and dies with E401 — and before failing it **deletes the
node_modules symlink** ("npm warn reify Removing non-directory"), leaving the
repo with no deps at all, so even vitest/vue-tsc/eslint stop working. Restore
with:

```
ln -s ../oc2plus-linecrm-frontend-backoffice/node_modules node_modules
```

Only the user can `npm login`; ask rather than retrying.

Related: [[reference-caddy-host-networking-gotcha]], [[feedback-verify-as-the-user-sees-it]]
