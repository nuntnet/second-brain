---
name: reference_go_toolchain_cache_damaged_after_fd_crash_use_gotoolchain_local
description: "After the 2026-11-02 file-table crash the cached go1.26.10 toolchain lost src/os/exec ('package os/exec is not in std'); build with GOTOOLCHAIN=local (Homebrew go1.26.1) until the user deletes the cache dir"
metadata:
  type: reference
---

Seen 2026-11-02 (AI-393), right after the machine-wide "too many open files in system" crash
([[reference_codegraph_watchers_exhaust_system_fds]]): every `go build` in a repo whose go.mod says
`toolchain go1.26.10` (chat-core, kit) failed inside the toolchain itself — `package os/exec is not in std`.
`~/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.26.10.darwin-arm64/src/os/exec` held only `internal/` and 2 test files.

**How to apply:** prefix go commands with `GOTOOLCHAIN=local` (Homebrew `/opt/homebrew/bin/go` = 1.26.1, satisfies
`go 1.26.0`). Don't delete the module-cache dir yourself (read-only, shared by every session) — tell the user the path;
removing it makes Go re-download a clean copy. A `[setup failed]` from `go test` after a crash is this, not your code.
