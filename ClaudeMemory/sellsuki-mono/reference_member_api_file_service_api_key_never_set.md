---
name: reference_member_api_file_service_api_key_never_set
description: "OC2Plus member-api sends FILE_SERVICE_API_KEY as X-User-Id; it is unset on staging AND dev and declared in no values file, so receipt upload 403s and surfaces as a generic 500"
metadata:
  type: reference
---

**Verified on staging 2026-09-25** from the pods, not inferred.

`POST /v1/me/point-claims` (submit a receipt) uploads the photo to file-service
**before** writing the `point_claim` row. The upload authenticates with:

```go
req.Header.Set("X-User-Id", h.apiKey)      // = FILE_SERVICE_API_KEY
req.Header.Set("X-User-Kind", "sellsuki.system")
```
`src/repository/file_service_repository/http.go:143`

🔴 `FILE_SERVICE_API_KEY` is declared `envDefault:""`
(`cmd/generics_server/main.go:71`) and **is not set on the staging pod nor the
dev pod** — 33 env vars, no `envFrom`. It appears in **none** of
`deployment/values-{base,development,staging,production}.yml`. Nothing fails to
boot; the header simply goes out empty.

So file-service's Keto subject is `sellsuki.system:` with an **empty id** and it
answers `403 permission_denied`. member-api then collapses that into
`500 unexpected_error` — the status code and the message both lie, which is why
this reads as a database or schema problem. The real line is
`use_case/point_claim.go:437`: `upload point claim receipt photo to file-service
failed`.

**A second, independent mismatch** — fixing the key alone is not enough. On
staging company `f2bf928c…` the permission IS granted:

- `sellsuki.filesystem.{create,delete,update,view}` → role `sellsuki.role:605106`
- role 605106 is assigned to **`sellsuki.user:d24813ec-c1a2-44be-a3f8-d76a4add2350`**
- `sellsuki.system:sellsuki-system` holds **zero** roles on staging (targeted
  `subject_id` query, not a page sample)

That is the kind trap in [[reference_file_service_keto_subject_kind]]: the grant
sits under `sellsuki.user`, the caller identifies as `sellsuki.system`, and Keto
treats those as different subjects.

**To actually fix it, all three, per environment:**
1. pick the service identity id (the local seed uses `sellsuki-system`)
2. assign a role holding `sellsuki.filesystem.*` to `sellsuki.system:<id>` —
   **per company**, see [[reference_file_service_grant_is_per_company]]
3. set `FILE_SERVICE_API_KEY=<id>` AND declare it in the repo's values files

Note `FILE_SERVICE_URL` *is* set on the pod (`http://file-service-svc.sellsuki`)
while appearing in no values file either — so this repo's deployment env does
not all live in the repo, unlike [[reference_ccs_deployment_values_live_in_the_repo]].
Reading the values files is not enough to know what the pod has; read the pod.

Supersedes the "all service URLs are empty" reading in
[[reference_member_api_service_urls_were_never_set]] — the URLs were fixed, the
API key never was.

Related: [[reference_envdefault_localhost_masks_missing_config]] (same class: a
default that makes a missing value look present) · [[project_oc4362_claim_cluster_gaps]]
