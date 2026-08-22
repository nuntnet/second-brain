---
name: sla-ladder-engine-state
description: SLA ladder epic (AI-159) shipped 2026-08-22 — MRs open (ai-agent !4 → chat-core !18 → FE !4 order), CronJob in-repo; QA time-warp + catch-up + migration-renumber gotchas
metadata: 
  node_type: memory
  type: project
  originSessionId: 9847fdca-bb8a-4e71-acd5-7642c3aec6d9
  modified: 2026-08-22T03:29:06.650Z
---

SLA ladder epic (AI-159) as of 2026-08-22: ALL cards implemented, **pushed, MRs open**: ai-agent `feat/e3-extraction` → [!4](https://gitlab.sellsuki.com/sellsuki/sellsuki/backend/sellsuki-ai-agent/-/merge_requests/4) (**merge first** — additive restricted_mode), chat-core `feature/AI-126-case-use-case` → [!18](https://gitlab.sellsuki.com/sellsuki/sellsuki/backend/sellsuki-chat-core/-/merge_requests/18), FE `feature/AI-122-flags-port` → [!4](https://gitlab.sellsuki.com/sellsuki/sellsuki/frontend/ai-chat-admin-frontend/-/merge_requests/4). All Jira cards In Review with closing comments. Post-merge suites: chat-core 2682/177 pkgs, ai-agent 340/14.

**Merge topology resolved this session:** origin/main (AI-18 vault + fix/AI-32) merged into chat-core branch; parallel session's S8 Track B (AI-37 Tier 1) also merged — **their migrations 0068-0071 renumbered to 0076-0079** (SLA ladder keeps 0068-0075, already applied to real DBs; Tier 1 unapplied per its own comment). chat-core still has stale open MRs !10/!11/!17 overlapping this line. CronJob is IN-REPO now (not SRE chart): `cmd/sla_ladder_sweep_trigger` + `deployment/values-sla-sweep-cron-*` + cron-arm64/amd64-th CI includes, gated by CI_JOB_ENABLE (off — activates with the service). ⚠️ SWEEP_URL assumes Service name `sellsuki-chat-core-svc` — verify on first deploy.

**Why:** future sessions must not re-implement or re-open MRs; migration numbers 0076-0079 are now claimed by Tier 1.

**How to apply:**
- QA time-warp: move `chat_sessions.last_customer_message_at` AND `sla_ladder_incidents.opened_at` AND `next_step_at` together (MarkStepFired guard requires equality).
- Catch-up gotcha: a step already past due AT OPEN gets skipped_catch_up — sweep once to open BEFORE walking time.
- Ladder config cache 10s TTL; SQL edits don't invalidate (API saves do). SQL JSON edits: temp file + `pg_read_file`.
- auto_return defaults DISABLED (migration 0075); per-workspace opt-in via settings page.
- Restricted-mode invariant: EVERY human_mode flip clears ai_restricted_mode; only auto_return re-sets after its own flip.
- ai-agent deploys BEFORE chat-core.
- go build in repo root drops the binary in cwd — one already got committed+reverted (9e5f974); it's gitignored now.

Related: [[ai-mvp-integration]], [[ai-merge-topology-risk]], [[parallel-sessions-duplicate-symbols]]
