---
name: reference-stuck-ci-job-holds-resource-group
description: A GitLab job stuck for want of a runner keeps holding its resource_group, so retrying it blocks the very fix meant to unstick it — cancel, don't retry
metadata:
  type: reference
---

Two separate layers make a merge-request pipeline in this group die without
running a line of code. Both were diagnosed and fixed on
`sellsuki-service-consent` on 2026-09-07; the same pair will recur elsewhere.

**Layer 1 — the tag points at a dead pool.** Jobs inherited from the SRE
template `pipelines/gitlab-ci-pipeline.golang-th.yml` ask for
`['amd','docker','general','sellsuki','staging-th']`. Every `Staging-th`
runner in this group is offline or stale while the two `Production` runners
are online, so those jobs are never claimed and GitLab eventually reports
`stuck_or_timeout_failure` — which reads on the MR as "the tests failed".
The fix is already in `sellsuki-ai-agent` and `sellsuki-chat-core`: override
the three template jobs (`unit_test_merge_request_amd`,
`build_merge_request_amd`, `code_analyse_merge_request_amd`) with
`tags: [docker]`. Copy that block; it is a workaround for infrastructure and
says so in its own comment. Compare `tag_list` between a repo whose CI passes
and one whose CI hangs — `glab api "projects/<path>/pipelines/<id>/jobs"` —
before assuming anything deeper.

**Layer 2 — the stuck job keeps its `resource_group`, and retrying makes it
worse.** These projects define resource groups (`test`, `build`, `analyse`).
A job that cannot be claimed still occupies its group, so a NEW pipeline
carrying the tag fix sits at `waiting_for_resource` — a different status from
`pending`, and easy to misread as "the infrastructure is broken deeper than
you thought". **Retrying the stuck job does not help: a retry reuses the
original commit's config, so it asks for the same dead pool and simply holds
the lock again.** Cancel it instead
(`glab api --method POST "projects/<path>/jobs/<id>/cancel"`); the waiting
pipeline flips to `pending` immediately and a Production runner picks it up.

Measured before/after on the same MR: `unit_test` went from 578s queued and
never claimed, to **success in 33s**. `code_analyse` still fails in ~10s for
an unrelated reason ([[reference-ai-agent-ci-gaps]] — `CODE_ANALYSE_SERVER_URL`
is unset group-wide) and is `allow_failure`, so the pipeline is green.

The lesson beyond GitLab: a queue status naming a *resource* rather than a
*runner* is telling you something is holding a lock, not that capacity is
missing. Look for what holds it before touching infrastructure.
