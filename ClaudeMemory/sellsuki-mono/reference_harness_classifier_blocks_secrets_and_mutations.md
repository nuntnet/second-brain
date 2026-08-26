---
name: reference_harness_classifier_blocks_secrets_and_mutations
description: "Claude Code's auto-mode classifier blocks certain Bash calls (secret-like env dumps, gRPC mutations) regardless of the user's chat approval — needs an interactive prompt or a settings.json rule, not a 'yes' in chat"
metadata:
  type: reference
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-26T04:23:42.958Z
---

Independent of anything said in chat, Claude Code's own auto-mode classifier can refuse specific Bash tool calls outright — e.g. `env | grep -iE 'postgres|db_'` (reads that look like credential extraction) and `grpcurl ... CreateRole/AssignRole` (state-mutating RPCs) both got blocked mid-session even after the user had already said "ok" / explicitly asked for the action.

**Why it matters:** a user's chat-level "yes, do it" does **not** unlock this — it's a harness-level gate, separate from conversational consent. Retrying the identical call sometimes succeeds on a later attempt (observed inconsistency, cause unclear — possibly session-level re-evaluation), but there's no reliable way to force it from inside the conversation. The only real unlocks are an interactive approval prompt appearing for the user, or the user adding a Bash permission rule in their Claude Code settings.

**How to apply:** when a command gets this specific denial (`"Permission for this action was denied by the Claude Code auto mode classifier"`), don't loop retrying it or try to rephrase/obfuscate the command to slip past the filter. Explain plainly to the user what the command was for, and either (a) hand them the exact command to run themselves in their own terminal, or (b) try once more later in the same session (sometimes clears) — but don't treat a single retry failure as fatal, and don't treat one success as proof the gate is now open for similar future calls.
