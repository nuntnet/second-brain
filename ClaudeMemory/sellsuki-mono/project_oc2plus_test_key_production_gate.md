---
name: project-oc2plus-test-key-production-gate
description: member-api TEST_KEY allowed member impersonation in production until 2026-08-28; gate is now AppConfig.TestModeMatches and SRE still needs to delete the secret value
metadata:
  type: project
---

`oc2plus-line-crm-service-member-api` shipped `TEST_KEY` into
`values-production.yml` from secret `oc2plus-crm-secret` with **no environment
gate**. Presenting it let a caller name any LINE user id and receive that
member's session with no LINE token (`auth.go`, `member.go`), and made OTP pass
on a fixed code (`member_liff.go`) — impersonation of any member of any company.

Fixed 2026-08-28 (MR !94 → develop, separate from feature MR !93):
- one chokepoint, `AppConfig.TestModeMatches`, is the only place allowed to
  compare the key
- gates on an **allowlist** of non-production stages, so `ENVIRONMENT` unset or
  misspelt (`prod`, `PRODUCTION`) fails closed
- constant-time compare; `TEST_KEY` removed from `values-production.yml`; a
  production pod carrying one logs an error at boot and still refuses it

⚠️ **Still outstanding**: SRE must delete the `TEST_KEY` value from the
production `oc2plus-crm-secret`. The code ignores it now, but the credential is
still sitting there.

Two tests were false green — they set `TestKey` without `Environment`, so after
the gate they silently took the LINE path and still passed because the mock
returned the same profile. Same shape as
[[reference_testify_permissive_default_wins]]. Staging/development test mode is
unchanged and still works.
