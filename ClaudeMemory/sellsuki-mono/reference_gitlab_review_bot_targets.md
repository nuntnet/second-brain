---
name: reference-gitlab-review-bot-targets
description: GitLab code-reviewer bot auto-reviews only MRs targeting main/develop — feature-branch-targeted MRs (e.g. CCS3 feature/provider) get no review and need a manual substitute pass
metadata:
  type: reference
---

The org's GitLab code-reviewer bot (`group_6_bot_*`, posts `<!-- code-reviewer-bot -->` notes, auto re-reviews on push) picks up MRs **only when the target branch is `main` or `develop`**. Observed 2026-07-23: every MR targeting main/develop got reviewed within minutes (!151/!148/!150/!89/!90/!86/!15/!39/!132), while CCS3 MRs targeting `feature/provider` (!251, !264) got zero bot notes.

**How to apply:** for MRs that must target a feature branch (CCS3 BOLA work lives on `feature/provider`), don't wait for the bot — run a substitute review yourself with the same lenses (auth/actor source, contract match, fail-open vs fail-closed, error mapping, tests) and post the disposition as an MR note (example: !264 note 83022, which caught a super_admin fail-open default).
