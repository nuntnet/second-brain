---
name: project_ai96_template_catalog_reality
description: AI-96's template families are mostly already reachable in chat-core — only follow-up has no HTTP surface, and the real blocker is port shape, not a missing backend
metadata:
  type: project
---

Verified against chat-core `main`, 2026-09-09. The admin console's
`createUnavailableKbTemplateAdapter` claimed there was no table or endpoint for
the ack/fallback/follow-up/canned template catalog. **That was wrong** — four of
five families have full read AND write paths:

| family | read | write |
|---|---|---|
| disclosure | `GET /v1/workspaces/:id` | `PATCH …/guardrail-config` |
| degrade (fallback) | `GET /v1/workspaces/:id` | `PATCH …/kill-switch` |
| ack | `GET …/sla-ladder-config` | `PATCH` same path |
| canned (tier 0) | `GET …/tier0-rules` | `POST` / `PATCH /:rule_id` |
| follow-up | **none** | **none** |

Two real gaps:

1. **Follow-up templates have no HTTP surface.** Config exists
   (`src/entity/followup/config.go`), the sweep reads it, but `route/followup`
   exposes only an internal `POST /sweep`. Small backend addition.
2. **Nothing matches `KbTemplatePort`'s shape.** `listTemplates` /
   `saveTemplateGroup` describe ONE aggregated catalog; the backend models each
   template as belonging to the feature that sends it — five configs, three
   write paths.

Gap 2 is a **product decision, not a missing endpoint**: (a) console reads/writes
the four existing per-config routes (no backend work beyond gap 1, matches the
backend's own model — recommended), or (b) chat-core grows an aggregating
endpoint that must stay in sync with five configs. Undecided as of 2026-09-09.
Correction landed as admin-FE !42.

**The pattern to remember**: an "unavailable adapter" doc comment in
`packages/adapters/src/unavailableAdapters.ts` is a claim about a backend that
nobody rechecks. `AdsSpendPort`'s identical note ("no ads-spend write endpoint
exists ANYWHERE in this monorepo yet") was stale for weeks after chat-core
shipped `GET/PUT /v1/workspaces/{id}/ads-spend`, which is what left the console
on a stub — found only by the completeness audit (adjudication #7), fixed in
admin-FE !41. That file's own doc says "Delete a factory the moment its backend
lands — the presence of an entry here is the todo list." Treat every entry as
unverified until checked against the route packages.

After !41, the only ports still on unavailable stubs in the LIVE wiring are
`kbTemplatePort` and `leadSuggestionPort` (AI-70, genuinely has no engine).

Related: [[reference_backend_without_frontend_is_invisible]],
[[reference_ai_chatbot_completeness_audit]].
