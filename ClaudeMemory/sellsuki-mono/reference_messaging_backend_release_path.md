---
name: reference_messaging_backend_release_path
description: "messaging-backend (GitLab project 568) release path: staging = manual deploy_staging_th_arm on the main pipeline (no -staging tags); prod = vX.Y.Z tag pipeline → deploy_production_amd; staging has no public ingress, so verifying needs tsh staging-th login"
metadata:
  node_type: memory
  type: reference
  originSessionId: b501fc3c-c299-49c5-bf29-a2137b682618
  modified: 2026-11-24T02:21:37.093Z
---

Checked 2026-11-24 (AI-439 / MR !118):
- No `vX.Y.Z-staging` tags exist. Staging deploy = `deploy_staging_th_arm` (manual) on the **main** branch pipeline, ns `sellsuki` on staging-th. Merged-to-main is not deployed until someone plays it (main 7d8a843 sat at `manual`).
- Prod = push tag `vX.Y.Z` (latest before this was v1.4.2) → jobs unit/integration/e2e/build_image_production_amd → `deploy_production_amd`.
- MR pipeline: unit_test_merge_request_amd + openapi_breaking_change run on `staging-th` pool (off 21:00–09:00 → `stuck_pending_no_matching_runners`); build_merge_request_amd only starts after tests pass.
- `/webhook/fb/*` has no public URL on staging (internal ingress only, SRE-1685-style Mapping needed), so a deauthorize probe needs `kubectl port-forward` — and the staging-th tsh session expires separately from production's; an unattended run cannot re-login (browser SSO). Check `tsh status` early.
Related: [[reference_messaging_backend]], [[reference_manual_staging_gate_silent_drift]].
