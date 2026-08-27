---
name: turbo-cache-crosssession-false-green
description: "Turborepo cache hits replay ANOTHER checkout's green logs — \"Tasks: 4 successful\" can mean nothing ran; force a re-run before trusting a verification"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 01c88554-c2ad-4379-b428-9806eeccfe92
  modified: 2026-08-27T17:46:49.831Z
---

Verifying a branch in a fresh worktree, `pnpm type-check` printed `Tasks: 4 successful, 4 total / Cached: 4 cached / Time: 80ms >>> FULL TURBO` — and the replayed log lines showed **paths from a different Claude session's scratchpad worktree** (`…/270e02aa-…/scratchpad/probe_codex_sprint10-admin-integrations/packages/…`).

So `tsc` never executed in the checkout being verified. The hash matched because turbo keys on input content, and the local cache dir is shared across every worktree of that repo on this machine. A green line was reported for work that did not happen here.

Why this is dangerous specifically: this is the exact shape of the failure it hides — parallel sessions each build a slightly different tree, one of them primes the cache, and a later session reads a green that belongs to someone else's inputs.

**Rule:** when the point of the command is *verification* (before claiming a branch compiles, before a report, before a merge decision), force the tasks to actually run:

```bash
pnpm exec turbo run type-check --force
```

Note the flag placement — `pnpm type-check -- --force` forwards `--force` to **tsc**, not turbo, and fails with `TS5093: Compiler option '--force' may only be used with '--build'`. Confirm the re-run with `Cached: 0 cached` in the summary.

Same caution applies to nx and any other content-hash task cache. Related: [[verify-as-the-user-sees-it]], [[parallel-sessions-duplicate-symbols]].
