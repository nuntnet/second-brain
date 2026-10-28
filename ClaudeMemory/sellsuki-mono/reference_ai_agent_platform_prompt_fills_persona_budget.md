---
name: reference_ai_agent_platform_prompt_fills_persona_budget
description: "In sellsuki-ai-agent the platform prompt text (language + live envelope + kbGroundingRules) was counted inside the workspace persona budget; on gpt-4o-mini (tier small, 3,000 runes) Thai platform text alone is ~2,900 runes, so any new rule turns ~50 tests red (jailbreak/persona) and truncates real personas. AI-339 branch fixes it on the live path."
metadata:
  node_type: memory
  type: reference
  originSessionId: b337846d-2ae6-45a6-9e1a-01f42711a62a
  modified: 2026-10-28T17:18:04.710Z
---

Found 2026-10-29 on AI-339. `generate_reply.go` prepends platform text to `cfg.SystemPrompt`, and `context_assembler` truncates the SUM to `LayerSystemPersona` (1500 tokens × tier scale; gpt-4o-mini = small = 0.5 → 750 tokens = 3,000 runes, runes/4). On `origin/main` (6394e73) a KB workspace on gpt-4o-mini loses almost all of its own persona in Thai.

Symptom when you add prompt text: `TestJailbreakSet_WorstCaseLeakIsPersonaTextOnly` (44 subtests), `TestKBGrounding_*`, `TestAssembledPrompt_*` fail with the persona marker missing and the prompt cut mid-sentence ("Use no…"). Also `testdata/system_prompt_without_context_budget.golden` pins the prompt and must be regenerated.

Fix on branch `fix/AI-339-no-substituted-numbers` (ai-agent e96ffc1): add the platform text's token count to the persona budget for non-Playground requests. Until that merges, any branch adding platform prompt text hits this. Related: [[reference_ai_chat_mr_and_jira_landing_recipe_2026_10_25]]
