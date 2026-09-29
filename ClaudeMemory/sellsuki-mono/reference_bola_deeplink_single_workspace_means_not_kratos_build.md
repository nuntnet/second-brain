---
name: reference_bola_deeplink_single_workspace_means_not_kratos_build
description: "BOLA \"You do not have access to that workspace / signs in to a single workspace\" on a ?workspace_id= link means the deployed FRONTEND BUILD is not VITE_AUTH_MODE=kratos, not a bad CCS binding; dev-th was local_jwt until BOLA-330"
metadata:
  node_type: memory
  type: reference
  originSessionId: ee31a72f-804c-423a-bf71-9bc2a7187b9c
  modified: 2026-09-29T14:19:28.355Z
---

The deep-link gate (`frontend/bola-frontend/src/lib/workspace-deep-link.ts`) returns `single-workspace-mode` whenever `getAuthMode() !== "kratos"`, **before** asking the server anything. So on `bola-web.dev-th.bearyweb.com` every `?workspace_id=` link (e.g. from OC2Plus/CCS3) is refused, whatever the company↔workspace binding says. The message text blames the account; the cause is the build.

Check without cluster access: fetch the deployed `assets/index-*.js` and grep `VITE_AUTH_MODE`. `{}.VITE_AUTH_MODE` = baked in empty = local_jwt. (Verified 2026-09-29 for dev-th; the Mahanakhon workspace `ed7e24aa-…` was the trigger.)

Fix is BOLA-330 (repo side, branches `feat/BOLA-330-dev-saas-mode` in bola-backend + bola-frontend, written 2026-09-29, not merged) blocked on BOLA-329 (SRE: Vault key, Oathkeeper rule, Kratos allowlist, `*.sellsuki.com` API host). Merge order 329 → backend → frontend; merging before the Vault key = pod never starts. Staging is already kratos mode.

Related: [[project_bola_auth_mode_deployment]], [[reference_company_workspace_link_lives_in_three_stores]].
