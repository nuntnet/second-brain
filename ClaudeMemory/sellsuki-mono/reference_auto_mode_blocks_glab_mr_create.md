---
name: reference_auto_mode_blocks_glab_mr_create
description: "Auto-mode classifier denies `glab api --method POST .../merge_requests` (External System Writes) even when GET and git push work; prepare the MR bodies as JSON + a one-shot script in the scratchpad and hand the user the command instead of retrying"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-11T08:15:48.177Z
---

**Opening MRs from this harness can be refused by the auto-mode permission classifier** ("External
System Writes"), independent of glab auth. Seen 2026-10-11 while sweeping five KB release branches:
`glab api` GET calls, `git push`, and the Jira MCP comment all went through; the five
`glab api --method POST projects/<id>/merge_requests` calls were denied (an earlier attempt with
`--input file.json` returned `Unauthenticated`, which was a different problem: glab did not attach
the token for `--input`; use `-f`/`-F` fields or let the user run it).

**Correction later the same day:** the persistent `Unauthenticated.` was NOT the classifier and not
the token — **glab resolves its host/token from the git remote of the cwd**. Every failing call ran
from the scratchpad (not a git repo); the same POST from inside `backend/sellsuki-chat-core` worked
first time. So: `cd` into any submodule with a gitlab.sellsuki.com remote (or pass
`--hostname gitlab.sellsuki.com`) before `glab api`. Also `-f description=@file` does not read the
file in this glab version (it stores the literal `@path`); send long bodies with `--input body.json`.
Python `urllib` is no fallback: no CA bundle (pass `cafile=/etc/ssl/cert.pem`) and
`glab config get token` prints 21 lines, not the token. Once the five MRs were open the classifier
let the POSTs through, so its denial earlier was a one-off, not a rule.

**How to apply:** when a task ends in "open N MRs", write each body to `<scratchpad>/mr/<project-id>.json`
(`source_branch`, `target_branch`, `title`, `description`, `remove_source_branch`, `squash:false`)
plus a `create-mrs.sh` that skips branches with an open MR, then give the user that one command in a
`bash` block. Do not loop on the denial. Same family as [[reference_dev_rps_membertier_missing_0025_present]]
(auto mode blocking rps grants). Project ids: chat-core 849, ai-agent 850, admin-fe 851, rag-core 787,
knowledge-connector 882.

**2026-10-13 update:** `glab api -X POST .../merge_requests -f ...` and `-X PUT .../merge_requests/N/merge`
both went through in the OC-4739 session (cwd = monorepo root, user had said "merge ได้เลย"). What the
classifier DID refuse that day: `kubectl set env` on a deployment the user had not named, creating a new
Pod on staging, reading Secret key names on staging, and a DELETE-less `kubectl apply` — each worked once
the user ran it in their own terminal (`! cmd`) or confirmed explicitly. So: try the glab call, and hand the
user a ready file + command only when it is actually denied.
