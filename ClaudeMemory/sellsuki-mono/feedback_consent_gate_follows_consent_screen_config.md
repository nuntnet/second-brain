---
name: feedback_consent_gate_follows_consent_screen_config
description: OC2Plus consent gating must follow each company's consent-screen configuration (require_accept vs optional/display) — never a blanket rule
metadata:
  type: feedback
---

User rule (2026-10-03, on OC-4361): **the consent gate must follow what the company configured on its consent screen — whether each document is required or not.** Gate on the policy decision (`allowed` / `acceptance_required` / `configuration_missing` from 3rdparty-api's policy), not on "has accepted everything".

**Why:** companies choose per document `require_accept` / `acknowledge_optional` / `display_only`; a blanket "must accept all" would block members of a company that required nothing, and "never block" would let required PDPA be bypassed by calling the API.

**How to apply:** any new member-api endpoint that reads the DB directly must call `requireConsentGate` (member-api `consent_policy.go`); optional/display-only pending documents never block; nothing bound blocks with 404 `consent_not_configured` (OC-4545). Test both modes. See [[project_oc2plus_consent_enforcement_model]].
