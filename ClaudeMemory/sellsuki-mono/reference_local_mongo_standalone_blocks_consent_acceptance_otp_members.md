---
name: local-mongo-standalone-blocks-consent-acceptance
description: "Local consent acceptance (OC-4340) needs 4 things or OTP members never pass the member-app consent gate (503) — Mongo replica set, write/outbox flags, 7 unique indexes, consentee sequence floor; fixed on this machine 2026-10-30"
metadata:
  node_type: memory
  type: reference
  originSessionId: 846eb860-82e8-4469-8af8-09ce62f37397
  modified: 2026-10-30T07:08:35.228Z
---

Symptom on local: member app shows "ตรวจสอบเอกสารสมาชิก … ลองใหม่"; `/v1/me/consent-policy` or the PUT `/acceptance` answers 503 `CONSENT_UNAVAILABLE` / consent :8096 `acceptance_unavailable`. Four separate causes, peeled one at a time on 2026-10-30:

1. **Mongo standalone** — `sellsuki-service-consent` reads/writes acceptance in transactions. Fixed in root `docker-compose.yml`: `command: --replSet rs0 --bind_ip_all` + healthcheck that runs `rs.initiate` with host `localhost:27017` (every service connects from the host). Data kept.
2. **Write flags off** — `CONSENT_ACCEPTANCE_WRITE_ENABLED` / `_OUTBOX_ENABLED` default false (dev values = true). Added to `scripts/setup.sh` template + `ensure_env_key` for existing `.env`.
3. **Missing unique indexes** — the service never creates them (`docs/acceptance-rollout.md` "index proposal"); `CheckReady` fails with "no matching unique index on consent for keys [id version]". Created the 7 from that table in db `develop-sellsuki-service-consent` by mongosh. NOT in any repo script yet.
4. **consentee sequence floor 29 < max id 9002** (seeded ids) — raised `sequence{type:consentee}` to 9002. Then `go run ./cmd/acceptance_preflight` → "Storage prerequisites passed" (that tool itself panics with a nil logger when a check fails — read the code at the failing line instead).

Consent logs are only in its tmux pane: find the server with `pgrep -fl "tmux -C -L .* -n consent"` then `tmux -L <name> capture-pane -p -J -t sellsuki-mono:consent -S -3000`. Same trick for any overmind proc.
