---
name: reference_bola_backend_gitignore_hides_cmd_bola_server
description: bola-backend .gitignore:41 is the bare pattern `bola_server` — it also ignores the DIRECTORY cmd/bola_server, so `git add` silently skips NEW files there (only a hint) while modified tracked files commit fine → a commit that does not build
metadata:
  type: reference
---

Hit 2026-09-30 on MR !205 (backend/bola-backend). `git check-ignore -v cmd/bola_server/quota_switch.go` → `.gitignore:41:bola_server`.

- The pattern was meant for the built binary but matches any path segment named `bola_server`, so the whole `cmd/bola_server/` directory is ignored for **untracked** files. Files already tracked (`main.go`, `helper.go`) are unaffected — which is exactly what hides it: the edited callers commit, the new callee does not.
- `git add <paths…>` with an ignored path only prints `hint: Use -f if you really want to add them` and stages everything else. In a chain (`git add … && …; git commit`) the commit still runs and succeeds without the new files. The first commit of !205 referenced an undefined function and did not build; a second commit added them with `git add -f`.
- Same trap class as the bare `.gitignore` in oc2plus member-api ([[reference_member_api_service_urls_were_never_set]]).

**How to apply:** after `git add` in bola-backend run `git status --short` and confirm every NEW file is `A `, and build the *committed* tree (fresh `git worktree add` of the pushed branch) before opening the MR — a build in the working directory hides this because the files exist on disk. Fix at the source is anchoring the rule to `/bola_server`; not done yet (separate card/MR). `git worktree remove` without `--force` still succeeds when the only leftovers are ignored files, so it will not warn you either.
