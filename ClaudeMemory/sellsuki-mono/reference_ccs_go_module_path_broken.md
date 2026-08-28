---
name: ccs-go-module-path-broken
description: "sellsuki-central-control-backend cannot be imported as a Go module — its go.mod declares share/backend/... while the repo lives at sellsuki/backend/...; vendor the proto instead"
metadata:
  type: reference
---

**STATUS 2026-08-28: fix open, not merged.** CCS
[MR !301](https://gitlab.sellsuki.com/sellsuki/sellsuki/backend/sellsuki-central-control-backend/-/merge_requests/301)
(`fix/go-module-path`, 238 files) declares the real path. Pipeline **success**
(unit_test 50s, build 1m54s; `code_analyse` is red on every branch including
`main` — SonarQube host unreachable from the runner, `allow_failure: true` in
the shared SRE template, so do not chase it). Proven end-to-end from a
throwaway module: `go get ...@fix/go-module-path` resolved and importing
`src/interface/grpc_server/company_service` compiled and ran. **Until it
merges, everything below still applies** — and after it merges, dropping a
vendored proto is a per-consumer decision, not automatic (chat-core's subset
deliberately omits `CompanyModel` fields 5-12 so resolving a display name
cannot also return address/tax/phone/email).

Two traps if regenerating protos in that repo: the old path also sits inside
each `.pb.go`'s serialized `rawDesc` behind **varint length prefixes**, so
text-editing it corrupts the descriptor and panics at proto registry init
(no compile error) — regenerate instead. And shortening `share`→`sellsuki`
reorders imports, so gofmt will flag files the rename touched.

Verified 2026-08-28. `backend/sellsuki-central-control-backend` lives at

    gitlab.sellsuki.com:sellsuki/sellsuki/backend/sellsuki-central-control-backend

but its `go.mod` line 1 declares

    module gitlab.sellsuki.com/sellsuki/share/backend/sellsuki-central-control-backend

That path does not exist, so `go list -m` / `go get` answers **404 Not Found**.
**No Go service can depend on CCS.** Confirmed by grep: no `go.mod` in the
monorepo names CCS, and there is no `replace` directive anywhere working around
it.

Contrast `sellsuki-role-and-permission-management-backend`, whose module path
matches its repo — which is why chat-core can import its generated gRPC package
directly (`src/repository/role_assignment_repository/grpc.go`).

**The house pattern for consuming CCS is to vendor a subset of the `.proto`**
with a local `go_package` and generate a client:
- `backend/space-go/src/repository/company_repository/` (3 RPCs)
- `backend/sellsuki-chat-core/src/repository/company_directory_repository/`
  (1 RPC, added by AI-94 — carries a `//go:generate` since it is chat-core's
  first generated protobuf)

A subset proto is wire-compatible as long as every kept field keeps its
**original field number** — proto3 decodes by number, not by completeness.

Two CCS facts found at the same time that matter more than the module path:
- **CCS's gRPC server registers NO auth interceptor** (only recovery +
  tracing in `src/interface/grpc_server/grpc_server.go`), and the `ApiKey`
  field its protos declare is never read on any path. space-go omits it too.
  CCS gRPC is trusted-network only.
- **`GetCompanyById` takes no caller identity** — it answers for any company id
  on the platform. Any service proxying it to a browser has built an IDOR
  unless it gates first.
- `ListCompanyThatHaveAccess` filters to grants of kind `sellsuki.company`
  only, so it returns an **empty list** for a user who holds only
  `sellsuki.chat_workspace` grants. See [[chatcore-company-list-derived-not-asked]].

Related: [[chatcore-is-the-admin-bff]], [[monorepo-no-origin]]
