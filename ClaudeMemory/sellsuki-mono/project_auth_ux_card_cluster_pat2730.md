---
name: auth-ux-card-cluster-pat2730
description: accounts.sellsuki (kratos-ui-go) auth/onboarding card cluster PAT-2730..2734 + PO decision 2026-09-25 — screens stay account-neutral, account-specific hints go in email only
metadata:
  type: project
---

Written 2026-09-25 after a user-driven review of signup → verify → login → recovery in `backend/kratos-ui-go`:

- **PAT-2731** [Security, High, blocks 2732] — signup via Kratos admin CreateIdentity (see [[kratos-ui-registration-bypasses-kratos-policy]]), OAuth secrets in git, enumeration, recovery patches traits pre-proof
- **PAT-2730** [Bug] — verify-email dead end; AC-C1–C6 = resend/countdown rules (server-side, 60s, resend = new flow), AC-O1–O7 = OTP input (cursor, backspace across boxes via keydown, one-time-code, auto-submit)
- **PAT-2732** [Story] — social ↔ password account linking, opt-in/opt-out, Settings "วิธีเข้าสู่ระบบ"; needs a ≤0.5d spike (email vs refid schemas)
- **PAT-2733** [Story] — recovery: email content per account type; PAT-2733 reuses PAT-2730 AC-C1–C6
- **PAT-2734** [Story] — i18n (none today; login.html matches Thai text to decide flow)

**PO decision (user, 2026-09-25): option (1)** — on screen, never reveal whether an email has an account or which method it uses (neutral message + show social buttons); say it specifically **in the email**. Identifier-first (option 2) explicitly deferred.

**Why:** balances UX with account-enumeration risk. **How to apply:** any new auth screen/copy in kratos-ui-go follows this; don't propose on-screen "this account uses Google" hints without revisiting the decision.
