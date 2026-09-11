---
name: reference-file-service-keto-subject-kind
description: file-service builds its Keto subject from X-User-Id AND X-User-Kind — a grant under the wrong kind 403s while the role sits right there
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-11T04:46:50.760Z
---

file-service checks `sellsuki.filesystem.*` on the company refID for **every**
route (`src/interface/fiber_server/route/file/route.go`), and it builds the Keto
subject from **both** headers — `helper/auth.go:11` reads `X-User-Kind`,
defaulting to `sellsuki.user`.

So a role granted to `sellsuki.user:<id>` is a **different subject** from
`sellsuki.system:<id>`. The failure is a bare `403 permission_denied` with no
hint the kind is what's wrong — the role exists, the id matches, and rps happily
accepts assignments under either kind.

The OC2Plus CRM services send `X-User-Kind: sellsuki.system`, so their grants
must be assigned to that kind. Measured on the running stack, same upload:
- no `X-User-Kind` → 200 (defaults to sellsuki.user)
- `sellsuki.user` → 200
- `sellsuki.system` → 403, until reassigned

**Two more traps in the same area:**
- `/upload/public` (route.go:50) **skips the permission check entirely** for
  whitelisted browser Origins. `/access/private` (route.go:223) has no such
  bypass. So "the logo upload works with a staff identity" proves nothing about
  a private read — and a server-to-server call has no Origin anyway.
- **Nothing in CCS or rps grants `sellsuki.filesystem.*` to anyone.** Any new
  service that talks to file-service needs an explicit grant per company, or it
  403s in every environment. `scripts/seed-dev.sh` now seeds it locally.
- `accessMode` values are **uppercase** (`ACCESS_URL`, `PRESIGNED_URL`, …) —
  lowercase gives a 400 that reads like a bad request, not a typo.

**Re-confirmed on staging 2026-09-11, and the whitelist does NOT include the CCS3 admin origin.** `WHITE_LIST` on staging (`deployment/values-staging.yml:45`) is `pis.staging*`, `api.staging*`, `reward.stg*.posh.oc2.plus`, `api.staging-th.ship-munk.com` — **not** `admin.staging-th.sellsuki.com`. `CheckWhiteList` is a `strings.Contains` over that list, so a fetch from the CCS3 admin app falls through to the permission check and gets `403 permission_denied` (measured: `POST /file/upload/public?refID=…` from `https://admin.staging-th.sellsuki.com` → 403). That also re-proves the "nobody is granted `sellsuki.filesystem.*`" point above, on staging, a year later.

⚠️ **Do not reach for the bypass.** Because the check is `Contains` on the `Origin` header, running the same fetch from a page on a whitelisted host (e.g. a `pis.staging-th.*` tab) skips the permission check entirely. That is circumventing an authorization control, not an integration path — get `sellsuki.filesystem.create` granted instead.

**`POST /upload/public` is the right endpoint for permanent public assets** (app-switcher tile icons, logos): multipart `file` + optional `context`/`metadata`, **omit `retention_days`** or the file auto-purges, and the response is `data.fileAccessURL` — a permanent public URL, which is what the `portal-app-registry` icon schema demands ("Never a pre-signed URL - they expire"). `/upload/private` is the wrong tool for icons for exactly that reason. `refID` only has to be non-empty (`helper/route_util.go:30-51` checks nil/blank, nothing more).

Related: [[reference-rps-identity-kind-must-be-prefixed]],
[[project-oc4362-claim-cluster-gaps]], [[reference_ccs_global_config_permission_gate]]
