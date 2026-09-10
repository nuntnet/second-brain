---
name: project_ccs_bola_provisioning_unwired
description: CCS→BOLA provisioning (BOLA-310 create workspace) has never worked in any environment — BOLA_API_BASE_URL/BOLA_SYSTEM_TOKEN are in no values file; found 2026-09-10
metadata:
  type: project
---

`POST /v1/companies/{id}/bola-workspaces` on sellsuki-central-control-backend
answers **500 in every deployed environment** and always has.

`AllocateBolaWorkspace` step 1 calls BOLA through `bola_api_client.CreateWorkspace`,
configured by `BOLA_API_BASE_URL`, which has
`envDefault:"http://localhost:8080"` (`cmd/generics_server/main.go:59`) and is
declared in **no** values file — staging, development, production or base, on any
branch. The call dials the CCS pod itself → connection refused → 500. Verified by
exec into the staging CCS pod: `localhost:8080/health` = refused,
`http://back-office-of-line-api-backend-svc.bola:80/health` = `{"status":"ok"}`.
The staging pod's env list has **18 vars, none containing "BOLA"**.

**The handshake:** CCS sends `X-System-Token: $BOLA_SYSTEM_TOKEN`; BOLA compares it
to its own `SYSTEM_ADMIN_TOKEN` in `middleware/machine_or_admin_guard.go` — empty →
falls through to the admin guard → **401 missing_credential**; wrong → **403
system_token_invalid**. BOLA's side *is* wired (secret ref on its pod). CCS's is not.

**Why the token could not just be added:** secrets are namespace-scoped. BOLA's lives
in `bola`; CCS runs in `sellsuki`, whose `sellsuki-central-common-secret` holds only
KAFKA_SERVERS, POSTGRES_DB_CENTRAL_SERVICE_URI, REDIS_ADDR, REDIS_PASSWORD. Referencing
a key that does not exist makes the pod fail to start with
`CreateContainerConfigError` — `bola-rgb-batch-cron` sits in exactly that state in the
staging cluster. So MR !324 ships the base URL live and the token as a **commented
block**; that converts the 500 into an honest 401, and finishing it needs someone to
add the `BOLA_SYSTEM_TOKEN` key and uncomment three lines.

Namespaces: staging `sellsuki` → `bola`; development `sellsuki-dev` → `bola-dev`;
production not touched (namespace unverified from this machine, and the CCS3 menu is
disabled on production).

**Pattern worth generalising:** this is the same defect as BOLA's own unset
`CCS_BASE_URL`, which the CCS3 AdminsPage comment already records ("the request reached
the backend, and an unset CCS_BASE_URL came back 501 … in every environment, because
CCS_BASE_URL was set in none of them"). Two services that call each other, each shipping
a client with a localhost default and no values entry. When wiring a new cross-service
call here, check the values files of the *caller* before believing the feature works.

Related: [[project_bola309_invite_via_ccs]] · [[reference_ccs3_frontend_facts]] ·
[[reference_flag_without_enforcement]]

## How these secrets are actually managed (2026-09-10)

`sellsuki-central-common-secret` and `bola-backoffice-secret` are **External Secrets
Operator** targets, not hand-made secrets. The tell is the annotation
`reconcile.external-secrets.io/data-hash` on the Secret. **`kubectl edit secret` is
reverted within the 300s refresh** — the value goes into Vault.

- ClusterSecretStore `vault-backend` -> `http://vault-server-active.system-security.svc.cluster.local:8200`, KV **v2**, no `path` set so the mount is the first path segment
- mount: **`kubernetes-secret`**
- each ExternalSecret uses `dataFrom.extract`, i.e. it pulls **every key** at one path,
  so adding a key in Vault needs **no ExternalSecret/YAML change**

| what | Vault path (mount `kubernetes-secret`) |
|---|---|
| CCS staging | `sellsuki/sellsuki-central-common-secret` |
| CCS development | `sellsuki-dev/sellsuki-central-common-secret` |
| BOLA staging (source of `SYSTEM_ADMIN_TOKEN`) | `bola/bola-backoffice-secret` |
| BOLA development | `bola-dev/bola-backoffice-secret` |

UI: `https://vault.internal.staging-th.sellsuki.com/ui/vault/secrets/kubernetes-secret/show/<ns>/<secret>`

**Two traps that actually cost time here:**
1. `sellsuki` and `sellsuki-dev` are sibling folders one character apart. A write to the
   dev path looks identical in the UI and syncs perfectly — into the wrong environment.
   Always check the breadcrumb, and verify per namespace with
   `kubectl -n <ns> get secret <name>` (compare the key list, never print values).
2. KV v2: **`vault kv put` replaces the whole secret**. Adding one key with `put` would
   wipe `POSTGRES_DB_CENTRAL_SERVICE_URI` and take CCS down. Use
   `vault kv patch -mount=kubernetes-secret <path> KEY=value`, or the UI's
   "Create new version", which carries the existing keys forward.

**How to tell "not synced yet" from "written to the wrong place":** read
`status.refreshTime` and `status.syncedResourceVersion` on the ExternalSecret. If
refreshTime is *after* the write and syncedResourceVersion is unchanged, the value is
not at that path — waiting longer will not help.
