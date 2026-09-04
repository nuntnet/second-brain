---
name: reference_gitlab_rules_silently_descope_jobs
description: "Adding one merge_request_event job to a .gitlab-ci.yml silently drops every rules-less job out of the MR pipeline; and rules without $CI_COMMIT_TAG kill tag pipelines entirely"
metadata:
  node_type: memory
  type: reference
---

Two failure modes of GitLab `rules:`, both hit in one sitting on
`ai-platform-kit-go` (2026-09-04). Neither produces a red pipeline — that is
the whole problem.

## 1. One MR-scoped job de-scopes every job that has no rules

A job with no `rules` / `only` / `except` keeps GitLab's implicit
`only: [branches, tags]`, which **excludes merge_request pipelines**. So the
moment you add a job with `- if: '$CI_PIPELINE_SOURCE == "merge_request_event"'`,
every other job in the file stops running in the MR's own pipeline.

What it looks like: **two pipelines per push**, both green. The branch pipeline
runs the old jobs, the MR pipeline runs only the new one — and the MR widget
gates on the MR pipeline. Measured: kit pipeline 57077 (push) ran `test`; 57078
(merge_request_event) ran only `dependency_scan_govulncheck`. The MR had no
tests and no coverage gate behind it and nothing said so.

Fix: give every job the same rules block.

## 2. Those same rules then delete the tag pipeline

`CI_COMMIT_BRANCH` is **unset on tag pipelines**. So

```yaml
rules:
  - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  - if: '$CI_COMMIT_BRANCH == "main"'
```

matches nothing on a tag, and the pipeline is created with zero jobs.

For a **library** this is worse than for a service: nothing deploys, every
consumer pins a `vX.Y.Z`, so the tag IS the artifact. Kit pipeline 56834 on ref
`v0.4.0` had run `test` with `tag=True` — after the rules landed, the next
release would have shipped unverified. Add `- if: '$CI_COMMIT_TAG'`.

Verify with `glab api "projects/:id/pipelines?per_page=5"` and read the
`source` column — two rows for one push is the tell for #1.

Related: [[reference-shared-library-skips-sre-template]], [[reference-ai-agent-ci-gaps]]
