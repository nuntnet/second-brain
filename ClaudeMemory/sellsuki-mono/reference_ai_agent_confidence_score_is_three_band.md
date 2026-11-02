---
name: reference_ai_agent_confidence_score_is_three_band
description: "sellsuki-ai-agent's hand-over confidence is escalation.Score, which returns only 0.1 / 0.35 / 0.95 — any workspace confidence_threshold in (0.35, 0.95] behaves identically; a card AC like '0.6 → 0.9 changes hand-over' is untestable"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0960115-b2a9-4901-82ed-4c5dd98097dd
  modified: 2026-11-02T11:30:34.898Z
---

Found on AI-389 (2026-11-02). `generate_reply.go` low-confidence gate is `escalation.IsLowConfidence(escalation.Score(body), cfg.ConfidenceThreshold)`, and `Score` (`src/entity/escalation/escalation.go`) is a heuristic with three bands: 0.1 (empty/too short), 0.35 (contains a hedge phrase like ไม่แน่ใจ / not sure), 0.95 (anything else). There is no other confidence source.

Consequence: the admin's "hand over when confidence below X" only changes behaviour when X crosses 0.1, 0.35 or 0.95. 0.6 and 0.9 do the same thing. Decision Inbox D-193 asks whether the console (AI-374 / G5) shows the slider as steps or a continuous score gets its own card.

**How to apply:** when writing or testing an AC about confidence thresholds, test across a real band boundary (e.g. 0.3 vs 0.6 on a hedged reply), not the card's literal numbers, and flag the wording. Related: [[feedback_never_cut_ai_reply_gates_observe_only]] (low-confidence escalation is a gate that stays).
