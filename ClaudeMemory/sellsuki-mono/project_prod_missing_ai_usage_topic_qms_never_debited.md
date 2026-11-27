---
name: project_prod_missing_ai_usage_topic_qms_never_debited
description: 2026-11-27 prod Kafka has no ai.usage.recorded topic → ai-agent publish fails before the QMS debit; 285 usage records (51 customer_reply) stuck in the spool since 11-16; knowledge-connector readiness flaps (1s probe)
metadata:
  node_type: memory
  type: project
  originSessionId: 58c6eca6-d6eb-446b-bf6a-a91ba8b63329
  modified: 2026-11-27T04:21:40.591Z
---

Observed read-only on production 2026-11-27 ~10:50 ICT:

- `ai.usage.recorded` (and `.dlq`) do not exist on the production broker (`kafka-topics --list` shows only `ai-chat.events.customer_erased`), and ai-agent sets `AllowAutoTopicCreation: false`. `usage_ledger_repository/recorder.go` publishes FIRST and returns on error, so QMS is never debited. The spool at `/var/lib/sellsuki-ai-agent/usage-spool` on `sellsuki-ai-agent-0` held 285 records (51 billable `customer_reply`, the rest embeddings and system calls, ~$0.096) from 2026-11-16 onward; replayed=0 every round. chat-core's analytics topics are also missing ("Unknown Topic Or Partition" for message, lead_transition and tier_decision).
- `knowledge-connector` (1 replica, sellsuki ns) readiness probe `/system/readiness` timeout 1s fails ~9.8k times in 7 days, so the pod drops out of the Service now and then and Drive import gets a 502 `google_connection_failed`.
- `/v1/companies/:id/usage/cost` and the dashboard 503 on prod = `usage_analytics_not_configured`, by design (values-production.yml, AI-356 Q7).
- 2026-11-26 21:46 ICT: QMS ClusterIP refused connections for ~2 seconds, causing a subscription 503 for 5 companies; it recovered on its own.

**Fixed 2026-11-27 11:00 ICT:** applied `docs/release/kafka/2026-11-27-ai-usage-topics.yaml` (replicas 3, 90-day retention). The first retry round replayed 27 records with retrying=0; the remaining 258 drain by ~17:03 ICT. The chat-core analytics topics file (`2026-11-27-ai-chat-event-topics.yaml`) has not been applied.

**Why:** a missing topic is silent apart from the spool gauge `llm_usage_spool_pending`. Quota is never consumed, so the 100-message trial cannot work once `QUOTA_PLAN_GATE_ENABLED` turns on.

**How to apply:** before you call ai-agent metering done in any environment, list topics on the broker. A KafkaTopic CR, or the staging runbook's topic list (`docs/staging-deployment-runbook.md:56`), is not proof. After the topic is created, the spool replays by itself; QMS dedupes on MessageRef. Related: [[project_ai_chat_dashboard_and_usage_need_data_pipeline_prod]]
