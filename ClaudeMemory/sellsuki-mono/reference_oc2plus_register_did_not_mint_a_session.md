---
name: reference-oc2plus-register-did-not-mint-a-session
description: "OC2Plus member registration returned a member with no session, so every web guest-first flow 401'd on the next request — invisible inside LIFF, fatal on the web"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f1e1e4b4-53e4-4459-99e1-ea7fc29faaa9
  modified: 2026-09-11T13:48:32.511Z
---

`POST /company/{slug}/members` (member-api) created or linked a member and set
**no cookie**. Only two endpoints ever minted `oc2plus_crm_session`:

- `POST /auth/login` — LINE access token (LIFF)
- `POST /company/{slug}/auth/login` — phone + OTP (OC-4348 web login)

Inside LINE this is invisible: the LIFF app logs in with its access token right
after registering. **On the web there is no token**, so OC-4465's guest-first
point claim — register mid-submit, then retry — got 401 on the retry. No
`point_claim` row, and the backoffice showed nothing at all. Not an error to
search for; an absence.

**The tell:** a `member` row exists with today's timestamp and `session` has
zero rows for it. Check both before suspecting the frontend.

Fixed 2026-09-11 (MR !108 → develop): registering mints the session, because
reaching the end of `UpsertMemberViaLiff` means `MarkUsedAtomically` consumed a
pre-verified OTP session for that phone — the identical proof `OTPLogin` takes,
so it grants nothing new. `issueMemberSession` + `setMemberSessionCookie` now
hold the one shape.

**Careful with the OTP:** `verifyOtpCore` only marks the session *pre-verified*;
`POST /members` is what **consumes** it (GETDEL). So you cannot "just call
/auth/login afterwards" to paper over this — the code is already spent by then.
That is why the fix had to be on the register path.

See also [[project_oc4348_web_otp_session_minter]],
[[reference_oc2plus_member_api_test_mode_login]],
[[reference_file_service_grant_is_per_company]].
