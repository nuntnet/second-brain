---
name: project-staging-deploy-target
description: "How WizemovesOS staging deploys: SRE shared frontend pipeline (S3 + CloudFront) + manual Supabase jobs; k8s/Teleport approach was dropped"
metadata:
  node_type: memory
  type: project
  originSessionId: 202c78fd-87b6-44c7-bd9e-ce11e2a73ba1
  modified: 2026-11-10T14:16:58.780Z
---

Sellsuki standard (found 2026-11-10 by reading `sellsuki/sre/deployment/pipeline-deployment` and oc2plus frontends): **frontends deploy as static sites to S3 + CloudFront** via the SRE template `pipelines/gitlab-ci-pipeline.generic-frontend-npm-th.yml`. App `.gitlab-ci.yml` just includes it and defines `.variables_export_{development,staging,production}_frontend` (CI_JOB_ENABLE, DOCKER_IMAGE, APP_URL, APP_ENV). Build = `npm install && npm run build:<APP_ENV>`; `main` -> staging with manual deploy; runner tag `staging-th`. k8s/Helm is only for backend services. Registry is `registry.fountain.sellsuki.com` (the GitLab Container Registry is not enabled on this GitLab).

WizemovesOS follows that: `.gitlab-ci.yml` includes the template, staging only, `BUILD_OUTPUT=./apps/web/dist`, `APP_URL=$STAGING_APP_URL` (deliberately a variable: the `prepare` job creates the S3 bucket + Cloudflare DNS from it). Backend is Supabase, deployed by manual jobs `deploy_staging_supabase_{db,functions}` (npx supabase@2). Docs: `docs/deploy-staging.md`. Work is on MR !55 (branch `chore/staging-deploy-readiness`).

An earlier attempt (Dockerfile + k8s manifest + Teleport tbot + kaniko) was built, then removed as unnecessary. Side effects of it that the user ran and may still need cleanup: namespace `wizemoveos-staging` (+ Role/RoleBinding `wizemoveos-deployer`) on the staging-th cluster, and GitLab CI vars `STAGING_TELEPORT_PROXY/CLUSTER/TOKEN`, `STAGING_KUBE_CLUSTER`.

SRE request filed: Jira **SRE-1690** (board https://sellsuki.atlassian.net/jira/software/projects/SRE/boards/44) asks SRE for the S3 bucket + CloudFront (SPA 403/404 -> /index.html) for proposed domain `wizemovesos.staging-th.sellsuki.com` (spelling is **wizemovesos**, from "Wizemoves OS"; the repo path `sellsuki/wizemoveos` and the k8s namespace `wizemoveos-staging` keep the older misspelling, do not copy it into domains/DNS/buckets), AWS/Cloudflare CI vars access, and mentions the stray k8s namespace. Format precedent: SRE-1685 (Task, `[Project] ...` title, lowercase labels, DoD checklist). Note the template's `prepare` job only creates the bucket + DNS; CloudFront is created by SRE by hand.

**Still open:** SRE must make AWS/Cloudflare CI variables (`STAGING_AWS_*`) available to `sellsuki/wizemoveos` (outside the `oc2plus` group) and agree the staging domain; Supabase staging project not created; pipeline never run on a real runner. `glab` token here can read project metadata but not repo files (use SSH `git clone --depth 1`); `glab` and `tsh` need the sandbox disabled for TLS.

See also [[sandbox-tooling-gotchas]].
