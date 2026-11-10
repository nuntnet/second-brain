---
name: project-staging-deploy-target
description: "WizemovesOS staging plan history and current interim choice (Vercel web + Supabase Cloud backend, MR !56); k8s and S3/CloudFront paths on hold"
metadata:
  node_type: memory
  type: project
  originSessionId: 202c78fd-87b6-44c7-bd9e-ce11e2a73ba1
  modified: 2026-11-10T15:28:11.475Z
---

**Always `git fetch` and read `origin/main` before designing deploy work.** The local checkout was 108 commits behind main in Nov 2026, which led to a wrong first design.

**Current interim plan (user, 2026-11-10, "ไปซักพัก"): web on Vercel + backend on Supabase Cloud.** Delivered in **MR !56** (branch `feat/staging-sre-s3-cloudfront`, name is stale): `vercel.json`, manual CI job `deploy_staging_web` (`vercel pull/build/deploy --prebuilt` from the runner, because Vercel's Git integration does not support self-managed GitLab), manual `deploy_staging_supabase` (team's `scripts/deploy/*`, operator-run: needs the Supabase CLI, interactive `migrate` confirmation), migration `0042_default_app_settings` (app_settings only existed in seed.sql and `useAppSettings` uses `.single()`), DNS spelling fix, `docs/deploy-staging.md`. Vercel progress (2026-11-10): the user chose to use their existing **Hobby** team `wizemovesmartechs-projects` temporarily (it also holds personal portfolio projects; Hobby is non-commercial per Vercel, so this is a stop-gap with test data only). Claude created the empty project `wizemovesos-staging` there (id `prj_PRFekB08grtcgachZ6lyRQ0qvpeF`, team id `team_7y0jFGSEaSSxCt3jXybGctlb`), turned off its "Improve models with this project's data" toggle (was on by default), and set GitLab CI vars `VERCEL_ORG_ID` and `VERCEL_PROJECT_ID` (non-secret, unprotected). Still missing: `VERCEL_TOKEN` (the user must create and enter it), Supabase project, `SUPABASE_*` vars. Boundaries followed: Claude never enters passwords, tokens/API keys or card data; the user does. Earlier status: nothing existed yet: no Vercel/Supabase accounts, projects or CI vars (`VERCEL_TOKEN/ORG_ID/PROJECT_ID`, `SUPABASE_URL_STAGING`, `SUPABASE_ANON_KEY_STAGING`, `SUPABASE_*` for the Supabase job); `vercel@latest` unpinned; pipeline never run. Open for the user/company: vendor approval, billing owner, PDPA/data residency; Vercel Hobby is non-commercial only (Pro ~$20/seat), Supabase Free pauses after a week and has no backups (Pro from $25/month).

**On hold, files kept:** (a) team's STORY-1301/1302 plan in `deploy/k8s` + `scripts/deploy/*` + `docs/DEPLOY-CHECKLIST.md`: self-hosted Supabase on EKS staging-th via ArgoCD/Helm/Vault, "preparation only", never applied; DB on a 20Gi gp3 PVC (+ pgsodium PVC, `Prune=false`), storage in versioned S3, backups = SRE EBS snapshots still unconfirmed. The web container pieces of it were deleted in MR !56 (restorable from git history). (b) web on S3 + CloudFront via the SRE template `generic-frontend-npm-th`; Jira **SRE-1690** asked SRE for it and was put on hold by a comment (card still open; domain proposed `wizemovesos.staging-th.sellsuki.com`). MR !55 (old stale-checkout attempt) is closed.

**Runners:** project has 5 group runners (3 `Staging-th`, 2 production), none accept untagged jobs, so main's old untagged CI always failed with `stuck_pending_no_matching_runners`; MR !56 pins `default: tags` to `docker,general,sellsuki,staging-th,amd`. The staging-th cluster and runners are switched off at night (user), so pipelines stay `pending` in the evening.

**Naming:** DNS/domain spelling is **wizemovesos** (from "Wizemoves OS", user instruction). Repo path `sellsuki/wizemoveos`, namespace `wizemoveos-staging`, npm scope `@wizemoveos/*` and the `wizemoveos://` scheme keep the old spelling; never copy it into hostnames. A namespace `wizemoveos-staging` + unused Role/RoleBinding `wizemoveos-deployer` were created on staging-th by the user's script early on (the Role/RoleBinding are unused; the namespace matches the k8s plan).

**Facts for testers:** no health endpoint (static SPA); login is Google @sellsuki.com only (also enforced in DB by migration 0040); no shared test user (dev@sellsuki.com and the dev bypass are local only); new users get role SALE.

See also [[sandbox-tooling-gotchas]].
