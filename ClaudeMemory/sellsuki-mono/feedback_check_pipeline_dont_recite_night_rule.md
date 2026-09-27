---
name: feedback_check_pipeline_dont_recite_night_rule
description: "shipping.md §9 (CI off 21:00–09:00) is a default, not a fact about tonight — I twice predicted red pipelines without looking; they were green. Check the pipeline before saying anything about it"
metadata:
  type: feedback
---

2026-09-19/20: I told the user every pipeline opened after 21:00 would fail with `stuck_or_timeout_failure`. When I finally looked, runners were up and all were green; the later failures that night were different causes (runner node "Not ready", DNS "Could not resolve host", "0/9 nodes available") — the cluster shrinking, not "CI is off". I had to correct myself in the ledger.

**Why:** reciting the rule sounded diligent and cost a wrong prediction the user acted on. Same shape as [[feedback_verify_absence_claims]].

**How to apply:** before stating a pipeline's state or predicting it, query it (`glab api projects/:id/pipelines/<id>/jobs`) and read the failing job's trace tail for the real cause. Infra failures → retry the failed job **by id** once CI is healthy (never the whole pipeline).
