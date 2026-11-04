---
name: reference-scheduled-task-cannot-merge-classifier-refuses
description: "An unattended scheduled-task run cannot merge MRs: the auto-mode classifier refuses the merge API as 'Merge Without Review' (2026-11-04), though the same merge from an interactive turn went through"
metadata:
  type: reference
---

On 2026-11-04 a one-time scheduled task (created with the user's explicit "merge in order and deploy staging") was refused at the first `glab mr merge` by the auto-mode classifier ("Merge Without Review"), so nothing landed; the same merges done from the live conversation an hour later succeeded.

**How to apply:** don't schedule merges for an unattended run. Schedule only reads/retries/reports, or tell the user the merge has to happen in a live turn. Related: [[reference-bola-jira-has-no-in-review-status-and-auto-mode-denies-merge]].
