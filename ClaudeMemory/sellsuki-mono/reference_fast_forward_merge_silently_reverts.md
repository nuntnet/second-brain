---
name: fast-forward-merge-silently-reverts
description: "When main is an ancestor of an MR's branch, the merge is a fast-forward: git reports no conflict and silently drops every fix the branch never picked up — check `merge-base --is-ancestor` and diff against the OTHER mainline before merging"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 270e02aa-6ae1-461a-ada0-6fb19a8e9e87
  modified: 2026-08-28T04:37:20.653Z
---

An MR whose source branch has `main` as an **ancestor** merges by fast-forward.
Git asks nothing, reports no conflict, and the target branch simply becomes the
branch tip — **including reverting anything the target had that the branch never
picked up.** No pipeline can see this: CI builds `main + branch`, never
"main after the FF, compared with the other mainline".

Hit on 2026-08-28 in `sellsuki-messaging-backend`, deciding the order of !14
(`main` ← `feature/mvp-media-and-secret-durability`) and !26 (`main` ←
`develop`). !14 was a fast-forward and was missing 16 `develop` commits:

| missing commit | what merging !14 first would have done |
|---|---|
| `cb154b2` remove unreachable break | `go vet` red on `main` |
| `a10be5f` MR !13 bot-review fixes 1–4 | chat module lands unreviewed |
| `5361d56` keep `.env` at main's values | `.env` overwritten with MVP-local values |
| `a4f0143` e2e `&&` word-split fix | CI breakage reinstated |

**The checks that settle it, in order:**

```bash
git rev-parse --git-path shallow     # must NOT exist, or ancestry lies
git merge-base --is-ancestor origin/main origin/<mr-branch>   # yes => fast-forward
git log --oneline origin/<mr-branch>..origin/<other-mainline> # what would be lost
git diff --name-only origin/main origin/<mr-branch> -- .env   # names only, never read
go vet ./...                          # on the branch tip, not on the merge
```

`go vet` on the **branch tip** is the cheap proof: it is exactly what `main`
becomes after the FF.

**Dry-run both orders — the conflict set is usually identical, and that is not
the deciding factor.** Here both orders hit the same three files
(`.env`, `.gitlab-ci.yml`, `fiber_server.go`). What differs is the *baseline the
resolver works from*: merge the promote-MR first and the conflicts get resolved
against a mainline that already has every fix; fast-forward first and they get
resolved against one that has already regressed.

**A conflict hunk can hold an already-fixed bug on the newer side.** The
`E2E_TEST_SCRIPT` hunk had `'make e2e-suite'` (the fix) on `develop` and the old
`'make a && make b && make c'` on the newer branch. "Take theirs" on the whole
file — the reflex, because the feature branch looks newer — reinstates the bug.
Resolve `.gitlab-ci.yml` hunk by hunk, never whole-file.

Related: [[reference_silent_semantic_merge_break]] (clean merge that fails to
compile — same family, different symptom), [[reference_messaging_backend_shared_repo_traps]]
(this repo's tracked `.env`), [[reference_submodules_are_shallow_clones]].
