---
name: reference_auto_mode_blocks_flag_flips
description: "Auto mode refuses Claude editing a deployed environment's feature flag (values-*.yml MCP_WRITE_ENABLED → true) — reason 'Feature Flag Writes'; plan the flip as the user's step, don't prepare it as a branch"
metadata:
  type: reference
---

2026-09-30, OC-4629: preparing a branch that set `MCP_WRITE_ENABLED: "true"`
(and added `oc2plus.write` to `OAUTH_SCOPES_SUPPORTED`) in
`backend/oc2plus-line-crm-service-backoffice-api/deployment/values-development.yml`
was denied by the auto-mode classifier with reason **[Feature Flag Writes]** —
even as an unpushed, un-MR'd branch. The denial covers the outcome, not the
command: don't retry it through another tool or split it up.

Same family as the other auto-mode limits in this workspace: Hydra client
PATCH, rps/Keto grants, applying migrations to dev/staging databases
([[reference_dev_rps_membertier_missing_0025_present]]).

**How to apply:** in a runbook, list the flag flip as a step the user does
(or the user runs Claude with a permission rule that allows it). Writing the
flag as `"false"` everywhere is fine — the denial is about turning it on.
