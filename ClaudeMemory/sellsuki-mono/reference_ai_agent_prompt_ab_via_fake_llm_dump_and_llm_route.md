---
name: reference_ai_agent_prompt_ab_via_fake_llm_dump_and_llm_route
description: "Measure an ai-agent prompt change on the live model without running a server from a worktree: dump GenerateReply's exact messages with the test fakeLLM, replay via the running agent's POST /internal/v1/llm (needs workspace_id+company_id body, X-Caller-Service header). Trap: history must END with the current bubble (chat-core's shape) or the baseline is wrong (AI-338: 0/6 vs real 3/5)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 102e0e35-0723-4b4d-b661-27917f5373ad
  modified: 2026-10-28T17:30:14.853Z
---

Used on AI-338 (2026-10-29) to A/B the facts instruction old vs branch.

- **Dump:** a throwaway `agent_test` file (never committed) builds `request()` + `CaseCheckpoint`, calls `agent.New(allowingBreaker(), &fakeLLM{...}, breaker.DefaultConfig).GenerateReply(...)` and writes `llm.lastMessages` to JSON. Run once on the branch, once with `git show HEAD:…/generate_reply.go` swapped in for the old version. Better still (what the verifier did): a forwarding LLM inside the test so the branch's own parser and guards run on the live answer.
- **Replay:** `POST http://127.0.0.1:8101/internal/v1/llm`, headers `X-Service-Token: local-dev-token` + `X-Caller-Service: <lowercase name>`; body needs `workspace_id`, `company_id`, `model`, `messages`, `max_tokens` (422 `field: workspace_id` otherwise). The live reply route is `/internal/v1/workspaces/{ws}/reply` (chat-core client `ai_agent_client/http.go:85`).
- **Trap — history shape:** chat-core sends the stored session, so `recent_turns` ends with the current customer bubble(s). A hand-built history without it changed the old prompt's result for a bare "27" from 3/5 to 0/6 and overstated a cause in the ledger. Build history the way chat-core does.
- **Trap — sandbox:** localhost, docker, the Go build cache and the overmind tmux socket (`/private/tmp/tmux-501/…`) need the sandbox off; TMPDIR differs between the two shells.
- Model-behaviour fixes regress in new places; each guard tweak needed an independent verifier pass (`verifier` agent resumed by SendMessage) — see [[reference_real_conversation_scenario_harness_lead_goal_step_2026_10_24]], [[feedback_reproduce_the_number_before_asking]].
