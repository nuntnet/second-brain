---
name: reference_bola_backfill_kratos_ids_cannot_run_as_documented
description: "bola-backend's cmd/backfill_kratos_ids is the designed fix for admins orphaned by the local_jwt→Kratos move, but its own documented runbook cannot work — the binary is not built into the image and it reads an env var name the pods do not have"
metadata:
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-29T11:44:32.467Z
---

**Verified 2026-09-29 by reading the repo; not yet fixed.**

`backend/bola-backend/cmd/backfill_kratos_ids` fills `admins.kratos_identity_id`
by looking each admin's email up in the Kratos admin API. It is the right tool
for admins left unreachable by the `local_jwt` → `AUTH_MODE=kratos` migration
(see [[project_bola_auth_mode_deployment]]), and it has a `--dry-run`.

Its own header documents:

```
kubectl exec -n bola <pod> -- /app/backfill_kratos_ids
```

**That command cannot work.** Two independent reasons:

1. **The binary is not in the image.** `Dockerfile:16` builds only
   `cmd/bola_server/*.go` into `/go/bin/app`, and line 26 copies just that to
   `/app/app`. There is no `/app/backfill_kratos_ids`.
2. **Wrong env var name.** The tool calls `mustEnv("ORY_KRATOS_ADMIN_URL")` —
   which exits when unset. The staging pod has `KRATOS_ADMIN_URL` and
   `ORY_KRATOS_PUBLIC_URL`; neither is the name it wants.

So the runbook in the file was written without being run. Treat the whole header
as unverified until someone fixes both and proves it end to end.

**To make it usable:** add it to the Dockerfile as a second binary, and have it
accept `KRATOS_ADMIN_URL` with `ORY_KRATOS_ADMIN_URL` as a fallback. Then
`--dry-run` first.

**Why this matters beyond one person:** OC-4591 counted 5,680 of 6,576 bindings
pointing at workspaces with no reachable admin
([[reference_bola_sweep_binds_without_an_owner]]). If that is still true, fixing
orphans one row at a time never finishes — this tool is the only thing that
scales, and it does not currently run.

Related: [[reference_bola_line_oa_channel_is_globally_unique]] — the usual way
this gets noticed is someone being unable to re-add their LINE OA.
