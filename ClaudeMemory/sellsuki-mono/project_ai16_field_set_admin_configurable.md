---
name: project-ai16-field-set-admin-configurable
description: Decision 2026-09-07 — the Customer Fact field set is a per-workspace feature admins configure themselves, not a compliance-approved fixed list; this is what unblocks AI-213
metadata:
  type: project
---

**Decided by the user on 2026-09-07**, answering the AI-16 AC8 "field-set
sign-off" gate: the Customer Fact field set **should be a feature an admin
configures per workspace**, not a fixed platform list waiting on
business/compliance approval.

**Why this unblocks a chain rather than starting a project:** the backend for
exactly this shape already shipped with AI-208 —
`backend/sellsuki-chat-core` has `src/repository/fact_schema_repository/` and
`GET` + `PUT /v1/workspaces/:workspace_id/fact-schema`. The per-workspace
store and write path exist on main.

What was still missing when the decision was made:
- **No admin UI.** `frontend/ai-chat-admin-frontend` has no fact-schema page
  (the only references are the Playbook editor sending `knownFactKeys`, which
  is the AI-213 gap itself), and no vendored OpenAPI/generated client for it.
- **Ports not wired.** `cmd/chat_core_server/main.go` calls
  `leadRuleUseCase.SetOptionalSources(nil, nil, consentChecker)` and
  `leadDataUseCase.SetOptionalSources(nil, consentChecker)`; the
  `FactSchemaSource` nil is a pure wiring gap now that an implementation
  exists.
- **Turning validation on fails closed on config that works today.**
  `lead_rule/repository.go`'s own comment says so: the default schema does not
  contain the fields existing pilot rules/fixtures use, so wiring the port
  without seeding/backfilling each workspace's schema from what its config
  actually references will reject live configuration.

Order that follows from the decision: admin UI + seed/backfill per workspace →
wire `FactSchemaSource` → AI-213 server-owned validation and the AC5 dropdown
(stop trusting caller-supplied `knownFactKeys`). See
[[project-preferred-language-is-constant-th]] for the other card whose AC
leaned on an assumption, and [[project-f08-fact-schema-versus-values]].
