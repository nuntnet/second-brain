---
name: reference_ccs_deployment_values_live_in_the_repo
description: sellsuki-central-control-backend ships its own deployment/values-{base,development,staging,production}.yml — an unset service URL is fixed by editing them, not by asking SRE; and their ports are each service's OWN defaults, never the monorepo Procfile's
metadata:
  type: reference
---

`backend/sellsuki-central-control-backend` carries its Helm values **in the
repo**:

```
deployment/values-base.yml
deployment/values-development.yml
deployment/values-staging.yml
deployment/values-production.yml
```

Each has a flat `env:` list of `- name: / value:` (or `secret: {name, key}` for
secrets, sourced by External Secrets Operator from Vault). **A missing service
URL is a code review away, not an SRE ticket.**

I missed this on 2026-09-17 by searching `values*.yaml` at `maxdepth 3` — the
files are `.yml` under `deployment/` — and told the user SRE had to set
`INVITATION_API_BASE_URL`. The dev team corrected it. **Search `deployment/`
explicitly before concluding a value lives outside a repo.**

## The ports in there are NOT the monorepo's

This is the part that misleads:

| | monorepo `Procfile` / port-map (LOCAL only) | `deployment/values-*.yml` (real) |
|---|---|---|
| rps HTTP | 9999 | **80** (`PORT: "80"` in rps's own values) |
| rps gRPC | 9998 | **50051** (its own Go default) |

So an in-cluster URL for rps is
`http://sellsuki-role-and-permission-management-backend-svc:80`, and the gRPC
address is `…-svc:50051`. Never carry a monorepo Procfile port into a values
file.

**Namespace suffix rule:** omit it when the callee shares CCS's namespace —
`ROLE_PERMISSION_GRPC_SERVER: sellsuki-…-svc:50051` has none. Add it when it does
not: consent is `.sellsuki` / `.sellsuki-dev`, BOLA is `.bola` / `.bola-dev`.

## The failure these files keep producing

`BOLA_API_BASE_URL` and `INVITATION_API_BASE_URL` both shipped **unset**, and
both have Go `envDefault`s pointing at localhost — so the pod dials itself, gets
connection refused, and one endpoint answers 500 everywhere while everything else
looks healthy. `BOLA_API_BASE_URL`'s own comment in values-staging.yml records
the first occurrence. When adding a service URL to CCS config, add it to all
three values files in the same change.

Related: [[project_ccs_bola_provisioning_unwired]] ·
[[reference_invite_accept_burns_the_code_on_grant_failure]]
