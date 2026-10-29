---
name: reference_pre_push_hook_false_ship_check_failed
description: "The monorepo pre-push hook can print \"ship-check FAILED\" when it actually crashed on its own bug (pre-push.sh:40 `args[@]: unbound variable`); warn mode still pushes — check the line above before believing it"
metadata:
  node_type: memory
  type: reference
  originSessionId: c4c0d24b-4441-439e-8950-fa80736ecd27
  modified: 2026-10-29T09:38:08.423Z
---

Seen 2026-10-29 pushing monorepo branch `docs/AI-346` (from the AI-49 line):
`plugins/sellsuki-workflow/scripts/pre-push.sh: line 40: args[@]: unbound variable` followed by
`[sellsuki-workflow] ship-check FAILED for the monorepo` and `mode=warn: pushing anyway`. The push landed.
`make ship-check` run directly was green ("nothing obviously missing").

**How to apply:** when that banner appears, look for the unbound-variable line first — if present, the check never ran;
run `make ship-check REPO=<path> BASE=origin/main` yourself and report that result, not the hook's. With
`SSK_HOOKS=enforce` this bug would block the push. Not yet fixed as of 2026-10-29 (unowned follow-up).
