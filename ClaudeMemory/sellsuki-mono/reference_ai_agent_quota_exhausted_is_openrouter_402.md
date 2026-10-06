---
name: reference_ai_agent_quota_exhausted_is_openrouter_402
description: AI agent 429 QUOTA_EXHAUSTED on local is NOT the company quota — the kit maps OpenRouter HTTP 402 (insufficient credits) to ErrQuotaExhausted; the shared OpenRouter key had $5 total credits. How to check and what it poisons.
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-06T13:35:37.524Z
---

`POST /internal/v1/llm` → `{"error_code":"QUOTA_EXHAUSTED","error":"the company is over its quota"}` (HTTP 429)
looks like the AI-82 package ceiling, but on 2026-10-06 it was **OpenRouter returning 402 Payment Required**:
`ai-platform-kit-go/llmclient/transport.go` `mapStatusError` maps 402 → `ErrQuotaExhausted`, and the route
renders that as QUOTA_EXHAUSTED. Redis had no `chat_core:quota:blocked:*` key and QMS had zero transactions,
which is how to tell the two apart quickly.

**Check in 10 seconds** (key from the running agent's env, never print it):
```bash
pid=$(pgrep -f "sellsuki-ai-agent/tmp/main" | head -1); KEY=$(ps eww -p $pid | tr ' ' '\n' | grep "^LLM_API_KEY=" | cut -d= -f2-)
curl -s https://openrouter.ai/api/v1/credits -H "Authorization: Bearer $KEY"   # {"data":{"total_credits":5,"total_usage":4.9967}}
```
Local key `LLM_PROVIDER_CODE=openrouter` had **$5 total credits**; the kb100 experiments spent them.

**Why it looks intermittent:** OpenRouter rejects a request when its estimated max cost exceeds remaining
credits, so a 35k-token sonnet call fails while a 20-token probe or a gpt-4o-mini judge call still passes.

**What it poisons:** Playground / chat replies degrade to `unavailable` (not an error), so a whole 100-question
baseline can be recorded as "unavailable 60/100" with zero request errors; LLM-judge grading rows come back
"ungraded". Always count `unavailable` statuses and check `/credits` before trusting a run.

**Fix:** top up credits on the OpenRouter account (user action, payment). Consider a dedicated key with a
limit per environment so one experiment cannot starve the shared local key.
