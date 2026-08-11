---
name: reference-rps-dual-mainline
description: "role-permission-service has TWO active mainlines (main→staging, develop→dev) — a feature landing on only one silently breaks the other env; migration numbering must dodge cross-branch collisions"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-11T12:58:48.307Z
---

`sellsuki-role-and-permission-management-backend` has **both `main` and `develop` as live deploy lines** (repo default = main). **Confirmed 2026-08-11: `main` builds STAGING (ns `sellsuki`), `develop` builds DEV (ns `sellsuki-dev`)** — both on the staging-th cluster. Staging deploy is a **manual gate job** (`deploy_staging_th_arm`); merging to main does NOT deploy until someone presses it. A failing `code_analyse_*` job does not block merge.

**How to apply:**
- **Every rps change needs BOTH MRs, opened together.** Cost of forgetting (hit 2026-08-11): the BOLA invite stack (!86/!90, `/internal/v1` + invitation metadata) landed on develop only, so staging had no internal API at all — BOLA staging invites failed closed with 503 and it took a full bisect (env → deploy → DNS → network → handler) to find. Fix was a `git merge origin/develop` sync MR (!96); the two mainlines' entire content diff was exactly that one feature, 18 files.
- After merging to main, **press the manual staging deploy job** — otherwise the pod keeps running the old image and you'll debug a fix that was never deployed.
- gormigrate keys migrations by string Id: number new migrations PAST the highest across BOTH branches (0014 was reserved for develop's invitation-metadata while main's seed was renumbered 0013→0015).
- `/internal/v1/*` endpoints have NO caller auth by design (cluster-internal; documented SECURITY BOUNDARY in route_internal.go) — hardening = lockstep change across CCS3 + bola-backend consumers.
- See [[reference-rps-is-system-role-trap]] before writing any role query in this service.
