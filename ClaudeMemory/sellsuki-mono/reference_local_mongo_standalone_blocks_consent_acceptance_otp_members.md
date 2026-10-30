---
name: local-mongo-standalone-blocks-consent-acceptance
description: Local MongoDB is standalone (no replica set) so consent-service acceptance read/write (Mongo transactions) always 503 — OTP members of any require_accept company can never pass the consent gate on local; not a product bug
metadata:
  node_type: memory
  type: reference
  originSessionId: 846eb860-82e8-4469-8af8-09ce62f37397
  modified: 2026-10-30T06:53:44.293Z
---

Found 2026-10-30. `sellsuki-mono-mongodb-1` (mongo:7) answers `rs.status()` → `NoReplicationEnabled`. `sellsuki-service-consent` reads/writes acceptance inside `session.WithTransaction` (`src/repository/acceptance_repository/read.go`), which needs a replica set, so `GET/PUT /consentee/acceptance` on :8096 always returns 503 `acceptance_unavailable`. 3rdparty-api passes it through as `/v1/me/consent-policy` 503 `CONSENT_UNAVAILABLE`; the member app shows "ตรวจสอบเอกสารสมาชิก … ลองใหม่".

So on local, member-app screens behind the consent gate (tier card, code redeem page, register fields…) cannot be verified for companies with `require_accept` bindings. Flipping `CONSENT_ACCEPTANCE_ENABLED` alone does not help. Real fix: single-node replica set (`--replSet rs0` + `rs.initiate()`) — ask first, it restarts the shared container. Workaround: binding `enforcement_mode=acknowledge_optional` with the 3rdparty flag off.

Also: overmind restart commands with `.overmind-*.sock` are relative — from a terminal not at the monorepo root they say "connection refused"; give absolute socket paths. `scripts/dev-ensure.sh check|ensure` lists/starts the expected per-socket instances. Related: [[reference_check_read_only_env_facts_yourself_user_does_not_know]].
