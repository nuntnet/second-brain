---
name: reference-go-rag-core-needs-go125-toolchain-sonic
description: "Go rag-core (project 864) does not compile on local go1.26 — sonic v1.14.1 \"undefined: GoMapIterator\"; run with GOTOOLCHAIN=go1.25.0"
metadata:
  node_type: memory
  type: reference
  originSessionId: d0da5cb3-1de8-4ef2-9e7c-5c5085687b5d
  modified: 2026-11-20T14:50:48.683Z
---

Go rag-core (`sellsuki/sellsuki/sellsuki-rag/rag-core`, project 864, not a submodule) pins `go 1.25.0`. The machine's default is go1.26.1, on which `github.com/bytedance/sonic@v1.14.1/internal/rt` fails with `undefined: GoMapIterator`. `go test` then prints only `FAIL … [build failed]`, which looks like a test failure but is not.

**How to apply:** `export GOTOOLCHAIN=go1.25.0 GOCACHE=$TMPDIR/gocache` before build/test in that repo (first run downloads the toolchain; cold build takes >5 min, run in background). Likely the same for any repo still on sonic <1.15. See [[feedback-prove-red-script-never-pipe-into-head]].
