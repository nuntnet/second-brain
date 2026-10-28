---
name: project_chat_core_not_deployed_to_production
description: "As of 2026-10-28 sellsuki-chat-core has NO production deployment (staging ns sellsuki only) — its cards' 'production step' is only values-production edits; don't list chat-core cards as waiting for a prod OK"
metadata:
  node_type: memory
  type: project
  originSessionId: 28036f73-9028-4b3a-98a8-7ab8e1f2c879
  modified: 2026-10-28T15:10:49.173Z
---

sellsuki-chat-core runs on staging (ns `sellsuki`) only; production has no chat-core deployment (recorded in docs/cards/AI-347.md "Ambiguities": "chat-core is not deployed there; values-production still edited for consistency").

**Why:** on 2026-10-28 I told the user "production for AI-351, AI-347, AI-350 waits on you"; they asked what that meant, and only AI-350 (data-pipeline consumer/dashboard + ai-agent, migration 0006 by hand) had a real prod step.

**How to apply:** before listing a chat-core card under "needs your production OK", check whether chat-core has reached production yet (`kubectl` prod context / values-production deploy jobs). Edit values-production anyway for consistency. Related: [[feedback_ask_decisions_with_full_context_not_bare_lists]], [[feedback_standing_approval_merge_deploy_staging_after_gates]].
