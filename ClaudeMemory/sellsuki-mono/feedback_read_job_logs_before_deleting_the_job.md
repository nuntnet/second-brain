---
name: read-job-logs-before-deleting-the-job
description: "2026-10-15 — a one-shot k8s Job (CCS _0026 on staging) reported 14 failures; I deleted the Job before reading them and the pod log went with it. gormigrate-backed migrations are then a no-op on re-run, so the failures are unrecoverable. Save the full log to a file FIRST, read the non-zero failure lines, only then delete."
metadata:
  node_type: memory
  type: feedback
  originSessionId: e5b57a1c-03b4-4838-8275-26b6728d42a6
  modified: 2026-10-15T16:00:06.086Z
---

A `kubectl delete job` cascades to the pod, and the pod log is the only record a headless
migration leaves. On 2026-10-15 the OC-3559 `_0026` Job ended `23,795 success, 14 failed`;
I had filtered the log for totals, deleted the Job, and only then noticed the 14. The
re-run at 22:58 printed "Total 0 items": gormigrate had recorded the id as applied, so
`MIGRATE_ONLY` skipped it. The 14 companies are unknown and need a per-company lookup.

**Why:** the summary line is not the evidence; the per-page `Errors: [...]` lines are.
A migration that tracks its own ledger cannot be re-run to recover them.

**How to apply:** for every one-shot Job: `kubectl logs job/<name> > <scratchpad>/<name>.log`
first, grep that file for `failed` lines that are not `0 failed` and for `Errors: [` that is
not `[]`, write the result into the ledger, and delete the Job last. Use
`ttlSecondsAfterFinished` instead of a manual delete when in doubt. The _0027 run minutes
earlier had 0 failures, which is why the filter looked safe — it was not.
Related: [[nothing-off-the-record]], [[reference-migration-files-are-not-applied-schema]].
