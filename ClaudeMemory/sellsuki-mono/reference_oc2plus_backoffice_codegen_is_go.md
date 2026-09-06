---
name: reference_oc2plus_backoffice_codegen_is_go
description: "OC2Plus member-api + backoffice-api HTTP codegen is a pure-Go generator (no TypeSpec/toolchain) — \"codegen blocked\" was wrong"
metadata: 
  node_type: memory
  type: reference
  originSessionId: b7f8ac01-fa37-4ae8-9246-e1a4f66c3859
  modified: 2026-09-06T17:17:52.007Z
---

The OC2Plus `oc2plus-line-crm-service-member-api` and `-backoffice-api` HTTP
interface codegen is **pure Go — needs NO external toolchain** (no `tsp`, no
installed `oapi-codegen`). Verified 2026-09-06.

- Source of truth = a **hand-authored** `src/interface/fiber_server/spec/v1.yaml`
  (editable — NOT the boilerplate CLAUDE.md's TypeSpec pipeline, which this repo
  doesn't use). `spec/v1/spec.gen.go` is the generated file (never edit).
- Regenerate with `make gen-http-fiber` = `go run cmd/generate_fiber_interface/main.go`.
  It uses `github.com/deepmap/oapi-codegen/v2` as a **Go module dependency**
  (library, via `codegen.Generate()`), so `go run` just works.

**Workflow to add/extend an HTTP endpoint:** edit `v1.yaml` → `go run
cmd/generate_fiber_interface/main.go` → implement the handler in `route/*.go`.

This corrects the recurring **"codegen-blocked" belief** that made me (and prior
context) call OC-4407 marketplace endpoint, OC-4421, OC-4335, OC-4363 etc.
un-buildable. They are buildable. I shipped OC-4407's marketplace channel this
way (member-api `POST /me/point-claims` + backoffice admin-list `marketplace`
field). See [[project_oc2plus_apikey_local_run]].
