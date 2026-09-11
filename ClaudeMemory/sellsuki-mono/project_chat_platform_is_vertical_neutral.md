---
name: project_chat_platform_is_vertical_neutral
description: The AI chat platform must not encode the insurance pilot's vertical — case types, guardrails and role names are generic by design; insurance-specific rules belong in opt-in presets and per-workspace config
metadata:
  type: project
---

Stated by the user 2026-09-11: **"ระบบ chat ของเราไม่ได้อยากให้ระบุว่าต้องใช้กับ
insurance business เราออกแบบมากลางๆ ใช้ได้กับหลายธุรกิจ"**

AI-57 and the surrounding cards were written against a Thai health-insurance
pilot, so insurance leaks into the code repeatedly. The rule: the platform floor
is what holds for a bookshop, a garage and a clinic alike; anything true only of
one vertical is a **preset or per-workspace config**, never a default.

**Corrections already made**
- `cmd/migration/migrations/0066_case_type_sales_lead` — the seeded case type
  went `insurance_lead` -> `sales_lead`. Its description is the canonical
  statement of the principle: a lead has the same shape whether the workspace
  sells insurance, cars or courses; what makes a workspace "insurance" lives in
  its fact schema, playbook, KB and persona.
- `src/entity/guardrail` (chat-core MR !84, 2026-09-11) —
  `DefaultHardRefusalKeywords` was the คปภ. set and was applied as an
  **unopt-outable floor**: `evaluatePreLLMGuardrailFunc` calls
  `DetectHardRefusal(body, nil)` before workspace config is loaded, and the
  workspace's own list is additive only. Claim-decision topics moved to
  `InsuranceHardRefusalKeywords` (opt-in via `Workspace.HardRefusalKeywords`).
  The concrete harm: **"เคลม" is not an insurance word in Thai** — it is warranty
  language for phones, car parts and appliances, so those shops had customer
  questions escalated to a human and never answered, with no way to turn it off.

**Still insurance-flavoured, comments only (code is generic)**
`src/entity/lead/default_template.go`, `src/entity/sheet_export/config.go`,
`src/entity/case_/subject.go`.

**Not a real signal:** rps roles 10/17/18/96 (`Insurance Provider Owner`,
`AI Chat Company Admin`, `Chat Workspace Operator`, `Chat workspace onboarding`)
are hand-made LOCAL TEST rows — every description says "(local test)" and no
source file contains those names. Do not cite them as product naming, and do not
propose reusing one as a canonical role.

**How to apply:** before adding to any platform default, ask whether a bookshop,
a garage and a clinic would all want it. If only one vertical would, it is a
preset. See [[reference_chatcore_is_the_admin_bff]] ·
[[project_ai_chat_platform_plan]].
