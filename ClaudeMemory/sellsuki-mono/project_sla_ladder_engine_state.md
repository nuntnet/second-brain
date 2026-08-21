---
name: sla-ladder-engine-state
description: SLA ladder epic (AI-159) fully implemented 2026-08-22 — all steps + restricted mode live-smoked; QA time-warp gotcha; catch-up gotcha
metadata: 
  node_type: memory
  type: project
  originSessionId: 9847fdca-bb8a-4e71-acd5-7642c3aec6d9
  modified: 2026-08-21T18:20:48.045Z
---

SLA ladder epic (AI-159) as of 2026-08-22: **ALL cards implemented locally, unpushed** — AI-161/163/165/171/170 plus the step cards AI-162 (ack), AI-172 (notify+unassign), AI-164 (restricted mode, cross-repo with sellsuki-ai-agent branch `feat/e3-extraction`), AI-167 (auto_return). chat-core rides `feature/AI-126-case-use-case`, FE `feature/AI-122-flags-port`. Migrations 0069–0075 applied locally. Everything live-smoked end to end incl. the full chain: auto_return → faq_only → price ask → holding message + hand back to humans.

**Why:** future sessions must not re-implement; remaining work is push/MR sequencing + Jira updates + K8s CronJob manifest (SRE chart, values in chat-core docs/sla-ladder-sweep.md).

**How to apply:**
- QA time-warp: move `chat_sessions.last_customer_message_at` AND `sla_ladder_incidents.opened_at` AND `next_step_at` together (MarkStepFired guard requires equality). The card's QA guide (move incident only) is wrong post-implementation.
- Catch-up gotcha when QA-ing: a step already past due AT OPEN gets skipped_catch_up — open the incident (sweep once) BEFORE walking time, or the step you want to see fire gets skipped honestly.
- Ladder config cache 10s TTL; SQL config edits don't invalidate (API saves do). Editing `workspaces.sla_ladder_config` by SQL: write the whole JSON via a temp file + `pg_read_file` — inline heredoc quoting breaks.
- auto_return defaults DISABLED inside the ladder (0075 flipped stored rows); enabling is per-workspace opt-in on top of the ladder switch.
- Restricted-mode invariant: EVERY human_mode flip clears ai_restricted_mode; only auto_return re-sets it after its own flip. `sla_ladder.no_open_incident` must match BEFORE the `sla_ladder.*` catch-all in FE mapCode.
- ai-agent deploys BEFORE chat-core (additive restricted_mode field).

Related: [[ai-mvp-integration]], [[ai-merge-topology-risk]]
