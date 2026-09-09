---
name: reference_local_stack_stale_branches_per_repo
description: The local stack silently runs whatever branch each submodule happens to be on, and the correct branch differs per repo — bola-backend=main, CCS=develop, CCS3-FE=an unmerged MR stack
metadata:
  type: reference
---

Symptoms reported as product bugs on 2026-09-09 that were all "this checkout is on an old branch":

- **suspend a BOLA workspace in CCS3 → BOLA still lets you in.** bola-backend was on an OC2Plus branch 834 lines behind `origin/main`, missing `526174e fix(BOLA-310)`. See [[reference_gorm_updates_drops_false]] — the old code was `Updates(&m)` with no `Select("*")`, so `is_active=false` (the zero value) was dropped while `updated_at` still changed. DB evidence: BOLA's row updated within 4 ms of CCS's and kept `is_active=t`.
- **"no add-user UI".** The Members panel exists in CCS3-FE MR **!438**; the running checkout was only at **!437** (BOLA-310).

**Each repo's correct line is different**, which is why "I'm on develop" proves nothing:

| repo | line that has the BOLA-309/31x work |
|---|---|
| bola-backend | `main` |
| sellsuki-central-control-backend | **`develop`** — `main` has 9 bola routes, develop has the full members + inviteFromBola set |
| sellsuki-company-management-frontend (CCS3 FE) | an **unmerged MR stack**: !437 → !438 → !439 → !441, tip `feat/bola-311-manage-invitations-ui` |

CCS trap: `origin/develop` still carries the wrong module path (`sellsuki/share/backend/...`); the fix lives only on `fix/go-module-path` (MR !301, unmerged — [[reference_ccs_go_module_path_broken]]). Verified `origin/develop` still `go build`s fine, so the wrong path only hurts consumers.

**How to check fast** rather than trusting the branch name: query the DB for the state the UI claims (`select is_active from workspaces`), and probe a route — a Fiber `Cannot POST /...` body means the route is absent, a JSON `error_code` means it exists and auth/permission refused ([[reference_ambiguous_404_fail_open]]).

Air survives a branch switch and rebuilds (~10-40 s) even when reparented to init after its overmind session died — see [[project_overmind_restart_quirk]].
