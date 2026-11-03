---
name: reference_lead_initial_stage_is_not_cold
description: "chat-core lead workflow — the IsInitial stage is \"cold\" only in the default template; the console mock's initial stage is \"new\". Never use is_initial as \"the Cold stage\" (hid every new lead, console !147, 2026-11-03)"
metadata:
  node_type: memory
  type: reference
  originSessionId: f746364c-7c1d-4189-93bc-f067e13be78d
  modified: 2026-11-03T13:39:26.973Z
---

chat-core `entity/lead/default_template.go` marks `cold` as `IsInitial`, and lead-from-chat creates into `InitialStage()`.
That made "initial = Cold" look safe. It is not: a workspace's config chooses its own first stage, and the console's
`mockLeadAdapter.ts` uses `new` ("ลูกค้าใหม่") as the initial stage. Console `87fd64d` hid the initial stage instead of the
literal `"cold"` (to answer review bot !147 finding 2, "Cold hard-coded"); the MR e2e lost every `new` lead
(`leads.spec` lead_demo_4 not found, axe a11y diffs on /leads). Reverted in `baa5b89`.

**How to apply:** "which stage is Cold" needs chat-core to say so (a stage kind/flag), not an inference from
`is_initial`. When a bot suggests replacing a literal with a derived value, check the derivation against the mock
config AND a non-default template before pushing — the vitest suite used the default template and passed.
Related: [[feedback_mock_mode_ui_review_check_real_adapter_before_calling_defect]].
