---
name: project-ai-platform-deploy-gating
description: "AI platform (chat-core) deploy is gated by SRE-side switches: CI_JOB_ENABLE project var, missing K8s secret, branch model develop→dev / main→staging; shared helm chart has no initContainers — migration wiring needs the SRE chart-ownership decision"
metadata: 
  node_type: memory
  type: project
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-05T16:08:08.449Z
---

Findings from AI-15 verification (2026-08-05), true for every service on the `sellsuki/sre/deployment/pipeline-deployment` golang-th template:

- **Deploy jobs only run when project CI variable `CI_JOB_ENABLE=true`** is set (rules.template.yml). Until SRE sets it, pushes to main run only `code_analyse` — this is why sellsuki-chat-core "deploys nothing" despite having full deploy jobs.
- **Branch → environment model**: `develop`/`develop-th` → development env; `main`/`master` (or `v*-staging` tag) → staging; prod via its own rules. chat-core has only `main`, so no dev-env deploys until a develop branch or model decision exists.
- **K8s Secret `sellsuki-chat-core-common-secret` does not exist yet** in any namespace — values files reference it (MR!7); someone with cluster/Vault access must create+populate per env before first deploy.
- **helm-generic-chart (SRE shared chart) has NO initContainers/Job support** — migration-on-deploy for chat-core (`/app/migrations` binary already in the image) cannot be wired without the SRE deploy-chart-ownership decision (same open item flagged on AI-15/AI-18; BOLA solved it with a separate SRE chart using an init container).
- Deploys use `helm upgrade --atomic --wait --history-max 2` → failed deploy auto-rolls-back; "rollback via pipeline" = re-run previous deploy job.
- gosec + govulncheck CI gates live in chat-core `.gitlab-ci.yml` (AI-15 AC3, MR!9); Go image family must stay on a **supported** Go line (1.24 fell out of support → stdlib CVEs were 9 of the 17 findings).

Related: [[project-ai-sprint234-autonomous-run]]
