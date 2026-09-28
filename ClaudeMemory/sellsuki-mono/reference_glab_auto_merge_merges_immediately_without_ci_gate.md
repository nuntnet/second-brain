---
name: glab-auto-merge-merges-immediately-without-ci-gate
description: "`glab mr merge --auto-merge` on OC2Plus 3rdparty-api merged at once while the pipeline was still running (the repo has no ci_must_pass); on backoffice-api the same call gets 405 while CI runs. It is not a \"merge when green\" button."
metadata:
  node_type: memory
  type: reference
  originSessionId: 78814507-bb88-4d16-a0fe-3ce06ba66f45
  modified: 2026-09-28T13:50:40.525Z
---

Seen 2026-09-28 on OC-3559:
- `backend/oc2plus-line-crm-service-3rdparty-api` !264: `glab mr merge 264 --auto-merge --yes` **merged immediately**, with head pipeline `running`. The repo does not enforce ci_must_pass, so "auto" has nothing to wait for.
- `backend/oc2plus-line-crm-service-backoffice-api` !647: the same command → `405 Method Not Allowed` while CI was running (the repo enforces `ci_must_pass`). Auto-merge could not be set.

**How to apply:** when the user says "merge when it's done", check the head pipeline status first and only run `glab mr merge` after it is `success`. Don't reach for `--auto-merge` as a shortcut on these repos. See [[feedback_list_open_mrs_before_opening_one]].
