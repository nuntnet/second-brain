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
