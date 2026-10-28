---
name: reference_ai_agent_needs_registered_company_charge_class_from_tier_path
description: "ai-agent /internal/v1/llm|embeddings refuses an empty/unregistered company (no \"platform\" scope); charge_class is a pipeline-generated column, so a caller can only mark a call via tier_path (feature is a closed kit enum)"
metadata:
  node_type: memory
  type: reference
  originSessionId: a7da547e-75b9-4bcd-99f8-99b884a2e2d5
  modified: 2026-10-28T18:31:21.143Z
---

Found on AI-348 (2026-10-29) while trying to book tenant-less rag-core calls as platform cost:

- ai-agent's scopes are only `workspace` and `company` (ai-platform-kit-go `llmclient/client.go`), and `verifyScope` (`sellsuki-ai-agent src/use_case/llm_gateway/service.go`) rejects a company that is not CCS-registered. **"No company on the wire" is impossible.** A tenant-less caller has to send some real company.
- `charge_class` is a STORED generated column in `data-pipeline/ai_chat_data_pipeline` (migration 0006, with a bridge copy in `internal/warehouse/migration.go`). It is derived from `billing_source`, `billable_ai_message`, `feature` and `tier_path`. Postgres refuses an explicit value. Changing the expression means drop + re-add, which rewrites `ai_usage_events`.
- `feature` is a closed enum in the kit (`llmclient/attribution.go`). A new value needs a kit + ai-agent release. `tier_path` is free-form and reaches the ledger verbatim on both routes, so it is the only mark a caller can add on its own (`kb_preview` → `admin_tool` already works this way).

**How to apply:** to classify a new kind of spend, stamp a `tier_path` value in the caller and add a WHEN to the pipeline's generated column. Don't propose sending an empty company or a new feature value as the quick fix. Related: [[project_oc2plus_saas_plan_epic_oc4763_decisions_2026_10_16]].
