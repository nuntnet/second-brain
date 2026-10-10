---
name: reference_k8s_env_value_dollar_dollar_becomes_dollar
description: "SQL passed to a one-shot kubectl pod through an env var loses `$$` (k8s turns `$$` into `$`) → plpgsql `DO $$` fails with a syntax error; use a `$x$` tag. Also glab mr create fails in the monorepo (glab-base remote exists) → create MRs with glab api"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7fb71c5f-6e71-4687-b858-9caa035598f1
  modified: 2026-10-10T15:03:59.031Z
---

Seen 2026-10-10 while deleting orphan rows on staging through a one-shot `kubectl run` pod (SQL passed in the `SQL` env var, then `printf '%s' "$SQL" | psql`).

Kubernetes expands env values: `$(VAR)` is substituted and `$$` is the escape for `$`. So `DO $$ ... $$` arrives as `DO $ ... $` and psql answers `syntax error at or near "$"`. With `ON_ERROR_STOP` everything before `BEGIN` has already run, but nothing gets written. **Use a named dollar-quote tag (`$x$ ... $x$`)** whenever SQL travels through a k8s env value.

Shell gotchas from the same session:
- zsh does not word-split `set -- $s`; use `set -- ${=s}`.
- `rtk` rewrites `curl` and truncates the response it stores (a 221-byte JSON with `...`). Use `rtk proxy curl ... -o file`.
- `glab mr create -R <repo>` run from the monorepo root fails with `remote glab-base already exists`. Create the MR with `glab api -X POST projects/<enc>/merge_requests -f source_branch=… -f target_branch=… -f title=… -f description=…` instead.

Related: [[reference_dev_cluster_access_and_readonly_db_pod]]
