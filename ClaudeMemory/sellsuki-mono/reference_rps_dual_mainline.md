---
name: reference-rps-dual-mainline
description: role-permission-service has TWO active mainlines (main AND develop) — MRs land on different lines and migration numbering must dodge cross-branch collisions
metadata:
  type: reference
---

`sellsuki-role-and-permission-management-backend` has **both `main` and `develop` as live target branches** (repo default = main). Observed 2026-07-23: MR !89 (BOLA role presets, migration 0013) targeted `main`; MR !86 (AllowNoRole, internal /internal/v1 create) and !90 (internal accept/roles-lookup/role-assignments + invitation metadata, migration 0014) targeted `develop`. Nobody confirmed which line deploys.

**How to apply:**
- Before opening an rps MR, ask/verify which branch is the deploy line — and make related MRs land on the SAME line.
- gormigrate keys migrations by string Id: number new migrations PAST the highest across BOTH branches (0014 was chosen while develop only had 0010, because main already had 0011–0013).
- `/internal/v1/*` endpoints have NO caller auth by design (cluster-internal; documented SECURITY BOUNDARY in route_internal.go) — hardening = lockstep change across CCS3 + bola-backend consumers.
