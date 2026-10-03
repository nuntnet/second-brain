---
name: reference_local_internal_key_exported_env_beats_dotenv
description: "Local 401 on backoffice→3rdparty internal calls was an INTERNAL_API_KEY exported into an overmind/tmux session, not a .env mismatch — godotenv never overrides process env"
metadata:
  node_type: memory
  type: reference
  originSessionId: 7fb71c5f-6e71-4687-b858-9caa035598f1
  modified: 2026-10-03T16:49:26.917Z
---

Found 2026-10-04 (OC-3559 test run). Backoffice "ปรับคะแนน" → 3rdparty `/internal/v1/.../point/adjust` answered 401 on local
although `backoffice/.env THIRDPARTY_INTERNAL_API_KEY` and `3rdparty/.env INTERNAL_API_KEY` were identical.

Cause: the `.overmind-oc2plus3rd.sock` session (`overmind start -f Procfile.oc2plus3rd`) was started from a shell that had
`INTERNAL_API_KEY` exported. tmux → air → binary inherit it, and `godotenv.Load()` does not override a variable already set,
so the process ran with a different key than its `.env`. member-api was started the same way (`THIRDPARTY_INTERNAL_API_KEY`).

How to check without printing secrets: compare `ps eww -p <pid>` env against the `.env` value in a script that prints only
lengths/equality; walk parent pids to see where it enters (here: the tmux server). `tmux -S <socket> show-environment -g`.

Fix used: aligned backoffice + 3rdparty `.env` to the running key (backups in that session's scratchpad), restarted backoffice
via air (no-diff rewrite of a .go file — air ignores .env). Related: [[reference_envdefault_localhost_masks_missing_config]] ·
[[reference_local_qa_run_facts]]
