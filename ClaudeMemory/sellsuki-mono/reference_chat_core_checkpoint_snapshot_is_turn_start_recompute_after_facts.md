---
name: reference-chat-core-checkpoint-snapshot-is-turn-start-recompute-after-facts
description: "chat-core's stored playbook checkpoint snapshot is computed at the START of a turn (TriggerMessage), before the agent's facts commit — anything reading Snapshot.Complete after the reply lags one turn unless it recomputes with TriggerFactWrite"
metadata:
  node_type: memory
  type: reference
  originSessionId: acb9d813-6c1d-48df-8dfa-e8f6f22c298b
  modified: 2026-11-02T15:24:36.311Z
---

In `backend/sellsuki-chat-core`, `reply_via_ai_agent.go` calls `ComputeCheckpoint(... TriggerMessage)` while building the agent context, i.e. before the agent's facts callback writes this turn's facts. The saved `checkpoint_snapshots` row therefore reflects facts as of the turn start. Code that runs after the reply and reads the snapshot (e.g. the Lead rule `playbook_complete` signal, AI-313) sees the customer's last answer only on their NEXT message.

Fix used on AI-313 (`feature/AI-313-lead-from-chat-v2`): recompute with `checkpoint.TriggerFactWrite` after the reply is stored, before reading. `TriggerFactWrite` existed but had no caller on main before that.

Also: when a playbook completes, `checkpoint.Compute` keeps the LAST step's id as `CurrentStepID` with `Complete: true`, so no `RecordStepChange` audit row is written for "became complete" — assert on the saved snapshot's `Complete`, not on a step change. Related: [[reference_ai_agent_confidence_score_is_three_band]].
