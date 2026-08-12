---
name: reference_local_identity_hardcode_loop
description: Local CCS3/invitation-SPA redirect loops come from hardcoded X-User-Id in committed .env.dev; fixed by Caddy forward_auth to Kratos whoami
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-12T15:29:31.528Z
---

Local `*.sellsuki.local` SPAs used to fake identity via `VITE_ADDITIONS_HEADERS`
in a **committed** `.env.dev` (CCS3 = `d46da77b…`, invitation SPA = `7cd97503…`).
Those identities do not exist in a freshly seeded local Kratos, so
`/v1/user/profile` → 500 → the app decides nobody is signed in → redirects to AMS
→ AMS sees a valid session → bounces back. Infinite loop, `?error=&return_to=`
growing each lap. Cost 3 separate debugging rounds in one day (2026-08-12).

Root cause: CCS reads **only** `X-User-Id`
(`src/interface/fiber_server/helper/auth.go` `GetIdentityFromHeader` — no cookie,
no session fallback). Production's AMS gateway sets it; locally nothing did.

Fix (committed to `Caddyfile`, `ccs.sellsuki.local`): the same `forward_auth`
pattern `oc2plus.sellsuki.local` already used —

```caddy
@has_user header X-User-Id *
handle @has_user { reverse_proxy localhost:8092 }   # explicit header still wins
handle {
  forward_auth localhost:4433 {
    uri /sessions/whoami
    copy_headers X-Kratos-Authenticated-Identity-Id>X-User-Id
  }
  request_header X-User-Kind sellsuki.user
  reverse_proxy localhost:8092
}
```

Consequences worth remembering:
- With this in place `VITE_ADDITIONS_HEADERS=` (empty) is correct locally. Both
  apps wrap `JSON.parse` in try/catch, and a missing value flips
  `withCredentials` on — which is what the cookie path needs.
- **Any test that depends on WHO you are was meaningless before this.** You were
  always the hardcoded user, so an invitation addressed to someone else appeared
  to work as the wrong person.
- `Caddyfile` still hardcodes `d46da77b…` for `pis.sellsuki.local` and
  `pis-app.sellsuki.local` — same trap, not yet converted.
- Local Kratos identities are only what `scripts/seed-dev.sh` created; check
  `select id, traits->>'email' from identities` in db `kratos` before blaming code.

Related: [[project_pis_frontend_local_testing]], [[feedback_verify_as_the_user_sees_it]]
