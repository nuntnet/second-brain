---
name: reference-file-service-keto-subject-kind
description: file-service builds its Keto subject from X-User-Id AND X-User-Kind — a grant under the wrong kind 403s while the role sits right there
metadata:
  type: reference
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

Related: [[reference-rps-identity-kind-must-be-prefixed]],
[[project-oc4362-claim-cluster-gaps]]
