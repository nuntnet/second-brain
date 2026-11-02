---
name: reference_chat_core_lead_vocabulary_guard_substring_trap
description: "chat-core src/guard/lead_boundary_test.go fails any .go line outside lead-owned packages containing \\bwarm|hot|cold|funnel\\b or the SUBSTRING stage/lifecycle, case-insensitive — so a tone code 'warm' and a test named TestAgent… ('stAge') both trip it"
metadata:
  node_type: memory
  type: reference
  originSessionId: 412dc360-5d42-492b-8f38-95d6097c2b92
  modified: 2026-11-02T04:47:25.895Z
---

Hit on AI-385 (2026-11-02). The guard scans every `.go` file under `src/entity`, `src/repository`, `src/use_case`, `src/interface` except lead-owned path segments (lead, lead_rule, lead_data, hot_lead_notify, …).

- `stage` is matched as a substring, case-insensitive: `TestAgentPersonaBudget` → "stAge"; comments count too.
- `warm`/`hot`/`cold` need word boundaries, so `ToneWarm` passes but the string `"warm"` and the label "Warm and caring" fail.

**How to apply:** a design value that happens to be a funnel word (the G2 persona tone "warm") needs a different code in chat-core (AI-385 used `caring`, D-152) or a deliberate guard exemption — say which in the ledger. Run `go test ./src/guard/` before pushing any chat-core change that names things.
