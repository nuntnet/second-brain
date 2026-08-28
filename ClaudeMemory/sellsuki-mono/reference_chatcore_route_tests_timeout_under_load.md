---
name: chatcore-route-tests-timeout-under-load
description: "chat-core's testcontainer route packages (workspace, case_, conversation) fail 7-10x slower under a full ./src/... run and hit Go's 10-minute timeout — always re-run the package alone before blaming a change"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 270e02aa-6ae1-461a-ada0-6fb19a8e9e87
  modified: 2026-08-28T14:11:18.577Z
---

`go test ./src/...` in `backend/sellsuki-chat-core` regularly reports one or
two FAILing packages that pass immediately when run alone. It is machine load,
not a regression — but it looks exactly like one, and re-deriving that costs
20+ minutes each time.

Measured on 2026-08-28, one machine, same day:

| package | in the full run | alone |
|---|---|---|
| `route/workspace` | FAIL 228s, then 330s | ok **6.3s** |
| `route/case_` | FAIL **603.0s** | ok **59.2s** |
| `route/conversation` | FAIL **602.1s** | ok **86.0s** |

**The 60x/603s tell.** ~600s is Go's default per-package test timeout: the
package was *killed*, not failed on an assertion. A `--- FAIL` line whose
duration is 10-120s on a testcontainer test is the same signal.

The failing test NAME changes every run — `TestPatchGuardrailConfig_StaleVersionIsAConflict`,
three `test: timeout error 1000ms` failures, `TestPatchWorkspace_RejectsBlankName`,
`TestPatchAIConfig_RefusesASystemPromptCarryingASecret`. A stable defect does
not move around like that.

**How to apply — the check that settles it, in order:**

```bash
go test -count=1 -timeout 25m ./src/interface/fiber_server/route/<pkg>/   # alone
git status --porcelain | grep <pkg>                                       # does the change even touch it?
```

Plus the cheapest control of all: **another branch's full run from the same
day.** If `./src/...` was green on a sibling branch an hour earlier on the same
machine, the delta is load or the branch — and the package list tells you
which.

**Do not skip the check when the package is topically related to the change.**
On the AI-115 merge resolution, the two timing out were `route/case_` and
`route/conversation` while the change touched case attribution — the one time
"probably flaky" would have been reckless. Running them alone took 2 minutes
and turned a guess into a fact.

Related: [[timing-dependent-concurrency-tests]],
[[turbo-cache-crosssession-false-green]] (the mirror: a false GREEN).
