---
name: reference_shared_library_skips_sre_template
description: "ai-platform-kit-go opted out of the SRE pipeline template (correctly — nothing to deploy) and thereby opted out of gosec+govulncheck; it carried 19 reachable CVEs with green pipelines"
metadata:
  node_type: memory
  type: reference
---

Measured 2026-09-04 on `backend/ai-platform-kit-go`:

```
govulncheck ./...  →  19 REACHABLE
  13 stdlib (golang:1.24 image, out of support)
   3 gofiber/fiber v2.52.6   — reachable via customerfacthttp.handlePatchSchema → Ctx.BodyParser
   1 google.golang.org/grpc v1.78.0
   2 golang.org/x/net v0.49.0
```

Every pipeline on `main` had been green for all of them.

**The mechanism, which generalises to any shared-library repo here:** deployable
repos get `gosec` and `govulncheck` from
`sellsuki/sre/deployment/pipeline-deployment`. A library correctly declines that
template (it has nothing to deploy) — and silently declines the security jobs
with it. Nothing replaces them, and no consuming service's scan covers the
library, because each scans its own tree.

Three services import this module (`sellsuki-ai-agent`, `sellsuki-chat-core`,
`sellsuki-messaging-backend`), so one CVE here reaches all three and is owned by
none of them.

Fixed in kit MR !16: fiber→2.52.12, grpc→1.82.1, x/net→0.57.0,
`toolchain go1.26.6` + `GOTOOLCHAIN` CI variable (the golang images bake
`GOTOOLCHAIN=local` and ignore the go.mod pin), image→golang:1.26, plus a
`dependency_scan_govulncheck` job. 19 → 0. **`gosec` is still absent.**

⚠️ The fix moves the module's `go` directive **1.24.0 → 1.25.0** — unavoidable,
`grpc@v1.82.1` and `x/net@v0.57.0` both declare 1.25. ai-agent and chat-core are
already there; **messaging-backend is still `go 1.24`** and must bump its own
directive before it can take the fix (it pins the pseudo-version
`v0.0.0-20260801124614-dd2d4312c72b`, so nothing forces it — it just keeps the
advisories).

Related: [[reference-gitlab-rules-silently-descope-jobs]]
