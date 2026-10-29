---
name: project_staging_ai_agent_usage_spool_stuck_poison_event_2026_10_29
description: "Staging ai-agent usage-spool replay fails every 5 min ('delivery not acknowledged') because QMS has no assign plan for company 018a163e-5e3f-444a-b4bd-96f73d42a312 / provider patona — unresolved as of 2026-10-29, clouds QMS-debit ACs (AI-71/133/350)"
metadata:
  node_type: memory
  type: project
  originSessionId: f4e6a7ce-8ac3-449f-a071-c829a2a83d39
  modified: 2026-10-29T07:07:14.838Z
---

Found 2026-10-29 by the G3 verifier: `sellsuki-ai-agent-0` logs `usage_spool: delivery not acknowledged` every 5 min since boot; quota-management-backend logs `record not found` on `gorm_assign_plans` for that company, provider `patona`. Reading, not proven: one poison spooled event. Also `AI_USAGE_ANALYTICS_SERVICE_TOKEN` is empty on staging ai-agent.

**Why:** makes "QMS debits unchanged" unprovable from staging logs.
**How to apply:** before trusting a metering AC on staging, check whether this is still happening; fix = assign that company a plan or drop the spool file (owner not yet decided). Sweep: docs/cards/sweeps/2026-10-29-verify-g3.md.
