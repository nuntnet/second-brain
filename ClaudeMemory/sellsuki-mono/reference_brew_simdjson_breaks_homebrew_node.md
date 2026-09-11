---
name: reference_brew_simdjson_breaks_homebrew_node
description: "A brew upgrade of simdjson (4.6.11 ships libsimdjson.33.dylib) leaves Homebrew node 25.8.0 linked against the deleted .30 and every node/npm/npx call aborts machine-wide; fix is brew reinstall node, and /usr/local/bin/node is only v16 so it is not a fallback for these repos"
metadata:
  node_type: memory
  type: reference
---

Happened mid-session 2026-09-11 on this machine, minutes after a `brew` upgrade —
nothing to do with any repo's code.

```
dyld[...]: Library not loaded: /opt/homebrew/opt/simdjson/lib/libsimdjson.30.dylib
  Referenced from: /opt/homebrew/Cellar/node/25.8.0/bin/node
```

`/opt/homebrew/Cellar/simdjson/4.6.11/lib/` contains `libsimdjson.33.dylib`; the
`.30` node was built against is gone. Every `node`, `npm`, `npx`, `vitest`,
`vue-tsc`, `eslint` call aborts (exit **134** = SIGABRT). `npm --version` fails
too, so the breakage is easy to misread as a repo/tooling problem.

**Diagnose in one line** — check the Cellar dir and compare to what node wants:
`ls /opt/homebrew/Cellar/simdjson/*/lib/`. A symlink timestamp on
`/opt/homebrew/opt/simdjson` that is *newer than* your last successful run is the
tell that the machine changed under you, not the code.

**Fix:** `brew reinstall node` (rebuilds against the present simdjson). It is the
user's toolchain — ask before running it rather than doing it silently.

**There is no usable local fallback:** `/usr/local/bin/node` is **v16.13.0**, far
below what these repos need (CI is node:22).

**Work around it instead of blocking:** push the branch and let GitLab CI verify
— the SRE template runs in a `node:22` container and is unaffected. Note the gap
honestly: `npm run lint` is *not* part of this project's CI jobs (only
`test:unit` + `check:boundaries` via `ci/unit-test-run.sh`), so lint cannot be
re-verified until the machine is fixed. See
[[reference_sre_test_job_slots_cannot_chain]].
