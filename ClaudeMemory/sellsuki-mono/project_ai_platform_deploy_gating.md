---
name: project-ai-platform-deploy-gating
description: "AI platform (chat-core) deploy is gated by SRE-side switches: CI_JOB_ENABLE project var, missing K8s secret, branch model develop→dev / main→staging; shared helm chart has no initContainers — migration wiring needs the SRE chart-ownership decision"
metadata: 
  node_type: memory
  type: project
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-27T18:02:48.413Z
---

Findings from AI-15 verification (2026-08-05), true for every service on the `sellsuki/sre/deployment/pipeline-deployment` golang-th template:

- **Deploy jobs only run when project CI variable `CI_JOB_ENABLE=true`** is set (rules.template.yml). Until SRE sets it, pushes to main run only `code_analyse` — this is why sellsuki-chat-core "deploys nothing" despite having full deploy jobs.
- **Branch → environment model**: `develop`/`develop-th` → development env; `main`/`master` (or `v*-staging` tag) → staging; prod via its own rules. chat-core has only `main`, so no dev-env deploys until a develop branch or model decision exists.
- **K8s Secret `sellsuki-chat-core-common-secret` does not exist yet** in any namespace — values files reference it (MR!7); someone with cluster/Vault access must create+populate per env before first deploy.
- **helm-generic-chart (SRE shared chart) has NO initContainers/Job support** — migration-on-deploy for chat-core (`/app/migrations` binary already in the image) cannot be wired without the SRE deploy-chart-ownership decision (same open item flagged on AI-15/AI-18; BOLA solved it with a separate SRE chart using an init container).
- Deploys use `helm upgrade --atomic --wait --history-max 2` → failed deploy auto-rolls-back; "rollback via pipeline" = re-run previous deploy job.

**Frontend template variant (found 2026-08-28 on `frontend/ai-chat-admin-frontend`, generic-frontend-npm-th):**

- There, `CI_JOB_ENABLE` is **committed in the repo's own `.gitlab-ci.yml`**, per environment, under
  `.variables_export_{production,staging,development}_frontend` — not an SRE-set project variable. All three
  were `'false'` with the comment *"flip once E8 is ready to actually deploy"*. So for frontends the switch is
  ours to flip, not a request to SRE (the k8s/S3 target still is).
- Consequence worth knowing: with every main-branch job gated off, GitLab creates **no pipeline object at all**.
  `glab api projects/:id/pipelines?ref=main` returned `0`, and `glab ci list --ref main` says
  *"No pipelines available"* — that reads like a tooling/auth error but is the literal truth: the app had
  **never been built or deployed by CI**, only MR-scoped `merge_request_event` pipelines had ever run.
  Check `?ref=main` count before assuming a repo has ever shipped.
- `glab ci list -b main` is not the flag — `-b` is parsed as a date. Use `--ref main` or the API.

⚠️ **Never flip a frontend `CI_JOB_ENABLE` to true without setting `APP_URL` in the SAME block.**
`templates/deployment.template.yml` runs `aws s3 sync --delete ${BUILD_OUTPUT} s3://${APP_URL}`, and
`templates/variables.template.yml` gives `.variables_export_staging_frontend` a **default
`APP_URL: "demo.staging.sellsuki.com"`** (dev: `demo.dev.sellsuki.com`, prod: `demo.sellsuki.com`).
A repo that overrides the block for `CI_JOB_ENABLE` but omits `APP_URL` can therefore `--delete`-sync
its build into a shared demo bucket, plus a CloudFront invalidation on that domain. Every other
frontend repo here (bola, company-management, sellercenter, pis) sets both together — treat the pair
as atomic. `ai-chat-admin-frontend` had the flag off and **no APP_URL at all** (no bucket ever
assigned), so "flip the flag and let it fail loudly" is NOT a safe probe there.

Branch→env gating lives in `templates/rules.template.yml`: staging = `main`/`master` (+ `v*-staging`
tag), development = `develop`/`develop-th`. A repo with only `main` can never run the dev jobs.
- gosec + govulncheck CI gates live in chat-core `.gitlab-ci.yml` (AI-15 AC3, MR!9); Go image family must stay on a **supported** Go line (1.24 fell out of support → stdlib CVEs were 9 of the 17 findings).

Related: [[project-ai-sprint234-autonomous-run]]
