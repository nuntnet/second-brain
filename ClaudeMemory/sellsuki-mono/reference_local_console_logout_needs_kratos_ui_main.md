---
name: reference-local-console-logout-needs-kratos-ui-main
description: "Local AI-chat console Log out fails because local kratos-ui-go runs an old feature branch that rewrites the JSON logout flow into a \"See Other\" stub; working logout is https://accounts.sellsuki.local/logout + confirm"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-17T17:24:27.278Z
---

2026-11-17: nunt could not log out of `https://chat.sellsuki.local`. Two causes, both local-only:
1. `apps/admin/.env` has `VITE_KRATOS_PUBLIC_URL=` empty → added `VITE_KRATOS_PUBLIC_URL=https://accounts.sellsuki.local` to gitignored `apps/admin/.env.development.local`.
2. `backend/kratos-ui-go` checkout is on `feature/self-register-app-launcher` (75 behind main, with uncommitted `.air.toml`, `kratos_schema_client/kratos.yml`). Its `InitBrowserFlow` replaces Kratos's JSON (`logout_url`/`logout_token`) with an HTML "See Other" body. Fixed on main in `9260d87` (in v1.7.0, which prod runs — prod logout is fine).

Working local logout (verified by replay with a real session: cookie cleared, chat-core 401): open `https://accounts.sellsuki.local/logout` and press "ออกจากระบบ". `/self-service/logout/browser` opened directly does NOT work. I first told nunt it was fixed before replaying — replay the real call with a session before saying a fix works.

**How to apply:** the console's own button stays broken locally until kratos-ui-go runs main's code; ask before moving that checkout (someone's branch with uncommitted work).
