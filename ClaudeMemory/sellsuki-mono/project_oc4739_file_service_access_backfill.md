---
name: project_oc4739_file_service_access_backfill
description: "OC-4739 (2026-10-13/14): CCS now grants the file-service identity a role at company creation + backfill cmd; dev done, staging backfill (14,749 companies) was in flight, plus a CCS crash to look at"
metadata:
  type: project
---

OC-4739 makes CCS create a company role "File Service Access" (`sellsuki.filesystem.create/view/list`) and
assign `sellsuki.system:sellsuki-system` (env `FILE_SERVICE_SYSTEM_ID`) at company creation, with
`/app/backfill-file-service-access` (dry-run default, idempotent) for existing ones. Merged: CCS !410 (main),
!411 (develop). Cause found via OC-3559 row 6: member-api/backoffice-api had no `FILE_SERVICE_API_KEY` and the
company had no such role, so receipt photo upload = 500, staff could not view the photo.

**State at 2026-10-14 ~19:50:** dev backfill done (4,885 companies, 0 errors, re-run `unchanged`). Staging has
**14,749** companies (not 4,171 — that figure is OC2Plus activation rows). Backfill must run as its OWN pod
(`scratchpad/bf-pod.yaml`: image of CCS, `nodeSelector/toleration node-role/application-arm`, rps
`sellsuki-role-and-permission-management-backend-svc:50051`): running it via `kubectl exec` in the CCS container
died whenever CCS restarted, and an unpinned pod landed on amd64 → `exec format error`. Still to do: dry-run
`unchanged` 14,749, merge member-api !236 + backoffice-api !731 (vault key set to `sellsuki-system` by the user),
deploy staging, send a real receipt on staging (AC8), Jira comment. Production backfill has no owner yet.

**Why:** staging/dev shut down 21:00–09:00; a run cut by that loses its result file.
**How to apply:** resume from `docs/cards/OC-4739.md` "Resume here". Also unresolved: staging CCS pod
restarted 3×, one with `panic: id (144) <= evictCount (222)` in x/net hpack inside the CCS process; dev CCS
restarted once too — not proven related to this change, needs an SRE/owner look.
