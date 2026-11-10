---
name: project-staging-deploy-target
description: "WizemovesOS staging: web on S3+CloudFront via SRE template (flag off), backend per deploy/k8s self-host plan; MR !56, Jira SRE-1690, hostname spelling wizemovesos"
metadata:
  node_type: memory
  type: project
  originSessionId: 202c78fd-87b6-44c7-bd9e-ce11e2a73ba1
  modified: 2026-11-10T14:46:29.042Z
---

**Always `git fetch` and read `origin/main` before designing deploy work.** The local checkout was 108 commits behind main in Nov 2026, which caused a wrong design (see below). `main` already holds the team's STORY-1301/1302 plan in `deploy/k8s/` + `scripts/deploy/*` + `docs/DEPLOY-CHECKLIST.md`: self-hosted Supabase on EKS `staging-th` via ArgoCD/Helm, Vault/ExternalSecrets, Google-only login, destination config repo `sellsuki/sre/configuration/wizemoveos`. It is "preparation only"; nothing was applied. `0040_auth_email_domain_guard` (DB-side @sellsuki.com check) already exists on main.

**Decision (user, 2026-11-10): the web is served from S3 + CloudFront** through the SRE shared frontend template (`sellsuki/sre/deployment/pipeline-deployment`, `generic-frontend-npm-th`): build `npm run build:<APP_ENV>`, `aws s3 sync` to a bucket named `APP_URL`, `prepare` job creates bucket + Cloudflare DNS, CloudFront is created by SRE by hand (custom errors 403/404 -> /index.html). Backend stays on the k8s plan. Registry is `registry.fountain.sellsuki.com`; GitLab Container Registry is not enabled.

Delivered in **MR !56** (branch `feat/staging-sre-s3-cloudfront`, on top of current main): migration `0042_default_app_settings` (+ SQL test; app_settings only existed in seed.sql, `useAppSettings` uses `.single()`), hostname spelling fix, CI integration. The web jobs are **off** (`CI_JOB_ENABLE: "false"` for staging) until SRE-1690 is done; then set CI vars `STAGING_APP_URL`, `SUPABASE_URL_STAGING`, `SUPABASE_ANON_KEY_STAGING` and flip the flag. Supabase operator jobs were renamed `deploy_staging_supabase` / `deploy_production_supabase` because the template owns `deploy_staging`/`deploy_production` (same-name jobs silently merge). The template's default runner tags are the legacy `staging` ones; CI pins `default: tags` to `staging-th`. MR !55 (old Docker/Teleport/S3 rewrite from the stale checkout) was closed as superseded.

**Jira SRE-1690** (board https://sellsuki.atlassian.net/jira/software/projects/SRE/boards/44) asks SRE for S3 bucket + CloudFront + AWS/Cloudflare CI vars + `staging-th` runner access for project `sellsuki/wizemoveos`, domain `wizemovesos.staging-th.sellsuki.com`. Runners: the project has 5 group runners (3 `Staging-th`, 2 production), online, not protected; the amd one has tags `docker,general,sellsuki,amd,staging-th`. NO runner accepts untagged jobs, so main's old untagged CI always failed with `stuck_pending_no_matching_runners` (that is why CI pins `default: tags`). The staging-th cluster and its runners are switched off at night (user, 2026-11-10; runner last contact was 21:03 ICT), so pipelines stay `pending` in the evening and run in office hours. Format precedent: SRE-1685.

**Naming:** DNS/domain spelling is **wizemovesos** (from "Wizemoves OS", user instruction). The repo path `sellsuki/wizemoveos`, namespace `wizemoveos-staging`, npm scope `@wizemoveos/*`, image name `wizemoveos-web` and the `wizemoveos://` scheme keep the old spelling; never copy it into hostnames.

**Open:** `deploy/k8s` `web.yaml` + `gateway.yaml` route the same web hostname to a container image and `build_web_image_staging` still exists; they conflict with the S3 choice (team decision pending). Namespace `wizemoveos-staging` + unused Role/RoleBinding `wizemoveos-deployer` were created on staging-th by the user's script early on; the namespace matches the k8s plan so do not delete it, only the Role/RoleBinding are unused. Pipeline never run on a real runner. Staging has no health endpoint and no test user (Google @sellsuki.com only; new users get role SALE).

See also [[sandbox-tooling-gotchas]].
