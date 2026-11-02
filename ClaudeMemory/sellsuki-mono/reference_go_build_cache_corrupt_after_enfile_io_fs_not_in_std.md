---
name: reference_go_build_cache_corrupt_after_enfile_io_fs_not_in_std
description: "After the machine's file table filled up (ENFILE), every `go build` failed with `package io/fs is not in std` / `os/exec is not in std` though the toolchain files were intact — the shared ~/Library/Caches/go-build was corrupted; a private GOCACHE builds clean"
metadata:
  node_type: memory
  type: reference
  originSessionId: c1216de2-2ddd-4868-a6b4-c29dbd97cd79
  modified: 2026-11-02T05:04:19.938Z
---

Seen 2026-11-02 (AI-379 side session, chat-core): after kern.num_files hit the limit and was cleared, the shared
build cache kept answering `.../toolchain@v0.0.1-go1.26.10.darwin-arm64/src/internal/filepathlite/path.go:14:2: package
io/fs is not in std` for every build, while `src/io/fs` existed on disk and descriptors were free again.

**How to apply:** `export GOCACHE=<session scratchpad>/gocache` for the session's go commands (first build ~1 min, then
normal), or prefix go commands with `GODEBUG=goindex=0` (AI-377 session, same day): the bad entry is Go's
module index of GOROOT inside GOCACHE, and that flag skips the index with no rebuild —
`GODEBUG=goindex=0 go list std 2>&1 | grep -c "not in std"` → 0 confirms it. Do not `go clean -cache` on your own while other sessions build in parallel — tell the user the shared cache is
corrupt and let them clean it. The 2026-11-02 occurrence was resolved by one `go clean -cache` (AI-385 session, 11:33);
the default cache built clean afterwards. Running air services may still hold binaries from before — check
`stat <repo>/tmp/main` before trusting a live check. Related: [[reference_codegraph_mcp_watchers_exhaust_macos_file_table]],
[[reference_sandbox_blocks_go_build_cache_and_bsd_sed_noop_mutation]].
