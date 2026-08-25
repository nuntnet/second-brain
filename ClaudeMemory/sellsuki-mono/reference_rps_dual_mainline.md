---
name: reference-rps-dual-mainline
description: "role-permission-service has TWO active mainlines (main→staging, develop→dev) — a feature landing on only one silently breaks the other env; migration numbering must dodge cross-branch collisions"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-25T11:00:00.000Z
---

`sellsuki-role-and-permission-management-backend` has **both `main` and `develop` as live deploy lines** (repo default = main). **Confirmed 2026-08-11: `main` builds STAGING (ns `sellsuki`), `develop` builds DEV (ns `sellsuki-dev`)** — both on the staging-th cluster. Staging deploy is a **manual gate job** (`deploy_staging_th_arm`); merging to main does NOT deploy until someone presses it. A failing `code_analyse_*` job does not block merge.

**How to apply:**
- **Every rps change needs BOTH MRs, opened together.** Cost of forgetting (hit 2026-08-11): the BOLA invite stack (!86/!90, `/internal/v1` + invitation metadata) landed on develop only, so staging had no internal API at all — BOLA staging invites failed closed with 503 and it took a full bisect (env → deploy → DNS → network → handler) to find. Fix was a `git merge origin/develop` sync MR (!96); the two mainlines' entire content diff was exactly that one feature, 18 files.
- After merging to main, **press the manual staging deploy job** — otherwise the pod keeps running the old image and you'll debug a fix that was never deployed.
- gormigrate keys migrations by string Id: number new migrations PAST the highest across BOTH branches (0014 was reserved for develop's invitation-metadata while main's seed was renumbered 0013→0015).
- `/internal/v1/*` endpoints have NO caller auth by design (cluster-internal; documented SECURITY BOUNDARY in route_internal.go) — hardening = lockstep change across CCS3 + bola-backend consumers.
- See [[reference-rps-is-system-role-trap]] before writing any role query in this service.

**No production deploy exists in this repo (verified 2026-08-25).** Job names across the last 20 pipelines are exactly `*_development_th_arm` and `*_staging_th_arm` — there is no production job of any kind, so `CI_JOB_ENABLE` for prod is unset here. Consequence when someone asks "will merging this reach production?": the answer from this repo's pipeline is **no**, and the pipeline also cannot tell you how rps *does* reach production — that is SRE's own chart, the same split as BOLA self-host's `bola-tenants` chart (see [[project_bola_deploy_topology]]). Getting a change to prod rps is a separate conversation with SRE, not a merge.

**Consumers pin `backend/entity` by tag, and merging to entity's main ships nothing.** entity has no `.gitlab-ci.yml` and has never run a pipeline — a tag is a plain git ref. Hit 2026-08-25: `sellsuki.chat_workspace` merged to entity main on 24 Aug but the newest tag (v0.30.0) was from 18 Aug, so the merged kind was invisible to every consumer and AI-182 looked done while nothing had changed. Always check `compare?from=<latest tag>&to=main` before treating an entity merge as delivered. See [[entity_lib_tenant_kinds]].

