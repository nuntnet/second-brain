---
name: reference-dead-staging-runner-tag
description: "CI jobs tagged 'staging' queue forever — that runner died ~Apr 2026; repos must include the -th SRE pipeline template variant"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T08:30:15.216Z
---

Runner tag **`staging` has no live runner** since the staging-th migration (~April 2026). Live runners: `staging-th` (5153 general/amd, 5154 arm, 5155 base) and `production`. Any repo still including the OLD SRE template `pipelines/gitlab-ci-pipeline.generic-frontend-npm.yml` (from `sellsuki/sre/deployment/pipeline-deployment`) produces jobs tagged `staging` that stay **pending forever** — no error, just silent queueing (sellsuki-invitation's main pipeline sat "running" from 6 Apr to 11 Aug unnoticed).

**Fix:** switch the include to the `-th` variant `gitlab-ci-pipeline.generic-frontend-npm-th.yml` (jobs extend `.staging_th_runner_amd_tags`; job structure otherwise identical — verified by job-name diff). Fixed for sellsuki-invitation in MR !15 (2026-08-11); sibling frontends like sellsuki-company-management-frontend already use it.

**Recurs for:** any dormant repo with the old include. Symptom signature = job pending with tags `['amd','docker','general','sellsuki','staging']` + zombie old pipelines stuck "running"/"waiting_for_resource" (cancel those to clear the queue).
