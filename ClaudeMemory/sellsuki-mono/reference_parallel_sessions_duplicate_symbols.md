---
name: reference-parallel-sessions-duplicate-symbols
description: Parallel Claude sessions write identically-named tests on different branches; git auto-merges them into duplicate declarations that only the compiler/vet catches
metadata:
  type: reference
---

Multiple Claude sessions work this repo at once, on different branches, from the same rules and the same findings. They therefore **independently write the same test with the same name**.

2026-08-15 both `main` and `fix/s8-review-fixes` had grown their own `TestGormPostgres_GetOrCreateOpen_SurvivesGenericPlanSwitch`. Merging produced **no conflict at all** — git textually interleaved them — and the result had two functions with one name, which does not compile. `go vet` was the only thing that caught it.

**A clean auto-merge is not evidence of a correct merge.** After any merge in this workspace, run `go build ./... && go vet ./... && go test ./...` before committing, even when git reported zero conflicts.

When deduplicating, compare the implementations rather than keeping whichever came first: in that case one version pinned `sqlDB.SetMaxOpenConns(1)` and the other relied on the pool "very likely" reusing a connection — the second could pass without ever reaching the bug it existed to catch. Keep the stronger one and fold the loser's provenance comment into it.

Practical note: `git worktree add` refuses a second worktree on a branch another session already has checked out. Work `--detach` at `origin/<branch>` and finish with `git push origin HEAD:<branch>`.

Related: [[feedback-parallel-sessions-git-safety]], [[reference-submodules-are-shallow-clones]]
