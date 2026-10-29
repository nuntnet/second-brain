---
name: reference-staging-fact-not-stored-diagnosis-chat-core
description: "Why a proposed customer fact was not stored on staging: chat-core keeps no proposed VALUE anywhere (span has key only, outcome counts go to Kafka); read schema/messages/customer_facts via psql in pg18 and infer; enum values must match exactly"
metadata:
  type: reference
---

Learned 2026-10-29 (AI-337, `โทร 0818800346` never stored as preferred_contact_channel).

- Run read-only psql with `kubectl -n datastore exec postgresql-pg18-0 -c postgresql -- psql -U postgres -d chat_core -At -c …` (exec into postgres was allowed; a chat-core replay POST is denied).
- `decision_traces` has `message_id`, not `message_ref`, and `fact_count` but no facts. `messages.body` (not `content`) holds the text. `customer_fact_schemas.fields` is jsonb: use `jsonb_array_elements(fields)` with `->>'key'`, `'type'`, `'enum_values'`, `'sensitive'`. `customer_facts` has key/value/source/updated_at.
- The proposed value is recorded nowhere. RecordExtractedFacts puts only `rejected_key` on the span, and the counts go to Kafka analytics. Infer the value from the reply, or replay locally (`/internal/v1/llm`, see [[reference_ai_agent_prompt_ab_via_fake_llm_dump_and_llm_route]]).
- The kit's enum check (`customerfact.ValidateFieldValue`) is exact and case-sensitive. Since ai-agent 7ec13e4/8bd07c3 (branch fix/AI-337-gate-counts-this-turn), ai-agent normalises enum facts to the listed value.
- ai-agent `keepRequestedFacts` allows only the step's MISSING slot keys. A schema field the step does not require can never be filled by the AI, so a stray amount lands in whatever amount-shaped slot is left. That is how `ทุน 1ล้าน` became budget_range.
