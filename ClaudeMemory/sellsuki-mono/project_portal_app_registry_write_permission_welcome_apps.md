---
name: project-portal-app-registry-write-permission-welcome-apps
description: Changing the live app-switcher registry (central-config portal-app-registry) needs configsystem.config.update on sellsuki.user:"" — dev has 4 holders, staging none; Kim's Welcome Apps editor exists on system-management develop only
metadata:
  node_type: memory
  type: project
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-27T17:10:58.669Z
---

The app switcher in every portal reads the GLOBAL central-config `portal-app-registry`. Migration 0003 seeds it only when absent, so a live change is a `PUT /configuration/portal-app-registry?userId=&location=` (version N+1, rollback by version).

Write permission (Keto read, 2026-11-27): `sellsuki.configsystem.config.update` on tenant `sellsuki.user:""`.
- **dev (`share-dev`)**: granted to `roles:sellsuki.system:#system_admin` — 4 holders (Chaiyakon, Kim, Chain, test account "suki space all permission"). nunt's own account gets 403, despite the "System admin" badge in CCS1's sidebar.
- **staging (`share`)**: 7 `system_admin` holders but the permission is granted only on `sellsuki.system:` → nobody can write the global registry via Keto (rps not checked).
- There is no system-admin role management UI in CCS1; the `system_admin` tuples were written by hand.

UI: **Welcome Apps** page `system.dev.sellsuki.com/config/welcome-apps` (form + JSON + version history/rollback) — built by Kim (kimzey / pan.nit) Jul–Aug 2026, merged to `sellsuki-system-management-frontend` **develop only**; not on staging/prod. nunt is talking to Kim about promoting it.

Pending (2026-11-28): the "Company Admin" tile for CCS3 (AI-49 ledger) — seed + icon merged, live registry still "Sellsuki" on all envs; script `docs/release/jobs/ai49-company-admin-tile-registry.js`.

**How to apply:** don't promise a registry change on staging/prod until the grant exists there; on dev ask one of the 4 holders or use the Welcome Apps page. A grant to `sellsuki.user:""` opens every global config — it is a security decision for nunt.
