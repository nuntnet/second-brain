---
name: sla-ladder-engine-state
description: AI-170 sweep engine done+smoke-verified 2026-08-21; remaining step cards order; QA time-warp gotcha
metadata: 
  node_type: memory
  type: project
  originSessionId: 9847fdca-bb8a-4e71-acd5-7642c3aec6d9
  modified: 2026-08-21T13:26:43.497Z
---

SLA ladder epic (AI-159) as of 2026-08-21: **AI-161/163/165/171/170 done locally, unpushed** — AI-170 (sweep engine) landed on chat-core `feature/AI-126-case-use-case` + FE snooze on `feature/AI-122-flags-port`, live-smoked end-to-end (open/catch-up/at-most-once fire/snooze/admin-reply resolve/disable-cancel).

**Remaining:** step handlers plug into `RegisterStepHandler` without touching the engine — AI-162 (ack) first, then AI-172 (notify+unassign), then AI-167 (auto_return) which is **blocked by AI-164** (restricted mode) per the card's own red banner. Deploy needs a K8s CronJob (`POST /internal/sla-ladder/sweep`, every minute, Forbid) that lives in the SRE chart, not this repo — see chat-core docs/sla-ladder-sweep.md.

**Why:** future sessions continuing the epic must not re-implement engine plumbing or fire auto_return before AI-164.

**How to apply:**
- QA time-warp gotcha: the card's QA guide says move `sla_ladder_incidents.opened_at` alone — WRONG post-implementation. MarkStepFired's guard requires `chat_sessions.last_customer_message_at = opened_at`, so shift BOTH (and `next_step_at`) together.
- Ladder config reads go through a 10s TTL cache (`ladderConfigCache`); API saves invalidate it, direct SQL edits don't — wait 10s before judging sweep behavior after a SQL config flip.
- `sellsuki.no_open_incident`-style codes: `sla_ladder.no_open_incident` must be matched BEFORE the `sla_ladder.*` validation catch-all in FE mapCode (already fixed, don't regress).

Related: [[ai-mvp-integration]], [[ai-merge-topology-risk]]
