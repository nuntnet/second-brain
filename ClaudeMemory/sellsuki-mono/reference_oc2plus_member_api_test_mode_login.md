---
name: reference-oc2plus-member-api-test-mode-login
description: how to mint a real member session on member-api locally without LINE, for end-to-end testing
metadata:
  type: reference
---

`oc2plus-line-crm-service-member-api` has a built-in test mode
(`src/use_case/auth.go:52`): when `TEST_KEY` is set and the request sends a
matching `X-TESTING-SECRET`, `POST /v1/auth/login` skips the real LINE profile
fetch and takes the profile from headers instead.

```
POST /v1/auth/login
X-TESTING-SECRET: <TEST_KEY>
X-LINE-USER-ID: <line_user_id>
X-LINE-DISPLAY-NAME: <name>
{"integration_id":"<uuid>","line_access_token":"unused"}
→ 204 + Set-Cookie: oc2plus_crm_session=...
```

It still needs real rows in `oc2plus_crm`, and this is the part that wastes
time — the login fails with a flat 401 if any are missing:
- `integration` (integration_id, company_id)
- `member_identity` (identity_id, line_user_id)
- `member` (member_id, identity_id, company_id)

`member_identity` has **no** `member_id` column — the link goes
`member.identity_id → member_identity.identity_id`.

These tables are **empty on a fresh local stack** — `seed-dev.sh` does not seed
them, so any member-facing endpoint is untestable until you insert them by hand.

Reaching the DB: this machine has **no `psql` binary** and docker is at
`/usr/local/bin/docker` (not `/usr/bin`). Use
`docker exec -i sellsuki_mono-postgres-1 psql -U postgres -d oc2plus_crm`.

Reading service logs: `overmind echo` **streams and never exits** — it will hang
a tool call. Capture the tmux pane instead; overmind uses a per-session socket:
```
SOCK=$(ps -ax -o command= | grep "tmux -C -L overmind-sellsuki-mono" | grep <svc> | sed -n 's/.*-L \([^ ]*\).*/\1/p')
tmux -L "$SOCK" capture-pane -p -t <svc> -S -200
```

Related: [[reference-local-kratos-identity-debugging]],
[[reference-file-service-keto-subject-kind]]
