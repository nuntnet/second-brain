---
name: reference_corrupt_go_build_cache_package_not_in_std
description: "`package io/fs is not in std (…/toolchain@…go1.26.10…/src/io/fs)` with the toolchain sources present = corrupt ~/Library/Caches/go-build (seen after the file table filled, 2026-11-02); build with GOCACHE=<scratchpad>/gocache instead of wiping the shared cache"
metadata:
  node_type: memory
  type: reference
  originSessionId: 3c6e4362-8fae-41c1-a18f-6e53fb5cee11
  modified: 2026-11-02T04:31:31.351Z
---

Seen on AI-382 (2026-11-02) right after the machine's file table filled up (codegraph watchers).

- Symptom: every `go build` in chat-core fails with `package io/fs is not in std` / `package os/exec is not in std`, pointing into `~/go/pkg/mod/golang.org/toolchain@v0.0.1-go1.26.10.darwin-arm64/src/...` — yet those directories and files are all on disk.
- Cause: bad entries in the SHARED build cache `~/Library/Caches/go-build`. The same build with `GOCACHE=<fresh dir>` succeeds.
- **How to apply:** `export GOCACHE=<session scratchpad>/gocache` for the session (first build is slow). Do not `go clean -cache` on your own — other sessions use it; tell the user, it is their call. Related: [[reference_codegraph_mcp_watchers_exhaust_macos_file_table]], [[reference_sandbox_blocks_go_build_cache_and_bsd_sed_noop_mutation]].
