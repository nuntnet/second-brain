---
name: reference_rps_proto_is_vendored_per_consumer
description: "Nine repos keep their OWN copy of the rps role_and_permission proto plus generated stubs — changing the proto in rps reaches none of them automatically; only CCS calls the invitation RPCs, and protobuf drops unknown fields silently so rps must deploy before any consumer that sends a new field"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-17T16:03:02.292Z
---

**Verified 2026-09-17 while doing PAT-2700.**

Consumers do **not** import a shared Go package for the rps gRPC client. Each one
vendors its own copy:

```
<repo>/src/repository/{role_and_permission,permission}_repository/proto/
    role_and_permission_service.proto
    role_and_permission_service.pb.go
    role_and_permission_service_grpc.pb.go
```

Nine repos hold a copy: `sellsuki-central-control-backend`, `management-backend`,
`i18n-management-backend`, `central-configuration-system`, `pis-api`,
`file-service`, `customer-book-backend-v2`, `sellsuki-inventory-management`,
`oc2plus-line-crm-service-backoffice-api`.

**So editing the proto in rps changes nothing anywhere else.** Each consumer that
needs the new field must regenerate its own copy. `make generate-grpc` in rps
touches only rps.

**Who actually needs to care:** only **CCS** calls the invitation RPCs (16 call
sites). The other eight use `CheckPermission` and friends, never touch the
`Invitation` message, and a stale copy there is harmless — do not regenerate
eight repos to "stay in sync", it is churn with no consumer.

**Checking cheaply** — do this instead of assuming:
```bash
git -C <repo> grep -l "CreateInvitation\|AcceptInvitation\|GetInvitation" -- 'src/**' | grep -v /proto/
```

**Drift is low but real.** CCS's copy differed from rps's only by block ordering
(`CountRoleMembers` moved), no semantic drift, as of 2026-09-17.

## 🔴 Deploy order: producer before consumer, always

**protobuf drops unknown fields silently.** No error on either side. If a consumer
deploys first and sends a field the server does not know, the value vanishes and
every log stays clean.

Concretely for PAT-2700/2701: if CCS ships before rps, the invited email is
dropped, `assertInvitationAddressedTo` sees no recipient, the invite silently
becomes an open link, and the UI still says "created successfully". There is no
failure to find.

So: **merge and deploy rps, confirm it is up, then ship the consumer.** This is
not a nice-to-have ordering — it is the only way the bug is detectable at all.
See [[reference_rps_dual_mainline]] for the two MRs rps itself needs.
