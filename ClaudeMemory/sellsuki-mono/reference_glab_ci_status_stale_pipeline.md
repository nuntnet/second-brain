---
name: glab-ci-status-stale-pipeline
description: "glab ci status --branch can report the PREVIOUS pipeline right after a push; glab ci trace hangs on a running job"
metadata:
  type: reference
---

Two `glab` traps that produced a wrong conclusion in one session:

1. **`glab ci status --branch <b>` returns the previous pipeline for a while after a push.** Right after pushing a fix, it reported the old pipeline's `failed` state, including `PIPELINE FINISHED: failed`. The new pipeline had not been created yet. **Always check the `SHA:` line against the commit you just pushed** before believing a result — a monitor keyed on branch alone will exit early on the old run's outcome.

2. **`glab ci trace <job>` follows a running job and never returns.** It hit the 120s command timeout. Use it only on a finished job; for a running pipeline use `glab ci status` or `glab ci get --pipeline-id N`.

`glab ci get --pipeline-id N` prints `status:` plus one line per job — the cheapest way to see the whole picture, and it distinguishes an `allow_failure` job (job `failed`, pipeline `success`) which `--branch` status alone makes look fatal.

3. **`glab ci get -b <branch>` breaks once TWO merge requests have used that branch name.** After an MR is merged and a second one is opened from a branch with the same name (e.g. reusing `feat/x` after `feat/x` was merged and the source branch deleted), it errors with *"failed to find associated merge request: merge request ID number required"* and lists both MRs — and an `until ! ... | grep -q running` watcher exits immediately on that error, so a pipeline that is still running looks finished. Resolve by MR id instead:

```bash
glab api "projects/:id/merge_requests/<iid>/pipelines"   # -> pipeline id + status
glab api "projects/:id/pipelines/<pipeline id>"          # poll this one
```

Also note `glab ci list` does not take `-b`/`--branch` (it parses the value as a timestamp) and `glab mr create` has no `--description-file` — pass `--description "$(cat file)"`.
