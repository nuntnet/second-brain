---
name: project_chat_core_unscheduled_sweeps_ai351
description: "chat-core has 9 \"CronJob calls this\" sweep endpoints but staging only runs sla-sweep + trace-retention crons; AI-351 adds rolling-summary only, the rest wait on Decision Inbox D-051 (2026-10-28)"
metadata:
  node_type: memory
  type: project
  originSessionId: c8122cba-dbd5-4605-8d20-45439f856a8f
  modified: 2026-10-28T08:26:08.024Z
---

Found 2026-10-28 (AI-347 verification): no conversation summary had ever appeared on staging because nothing calls the sweeps. Staging ns `sellsuki` CronJobs = `sellsuki-chat-core-sla-sweep-cron` (every minute, POST /internal/sla-ladder/sweep) and `sellsuki-chat-core-trace-retention-cron` only.

Unscheduled: rolling-summary + auto-close (AI-45), follow-up (AI-42), lead-reminders (AI-69), lead-rules auto-degrade (AI-55), sheet-export (AI-65), hot-lead-notify scan/send (AI-68), form-watcher detect. No outside caller in any submodule; n8n unchecked (MCP 502).

AI-351 (branch `fix/AI-351-rolling-summary-sweep-cron`) adds `POST /internal/session-summary/rolling-summary-sweep` (all active workspaces, keyset paged) + `values-rolling-summary-cron-*` every 10 min. Everything else stays off until the PO answers **D-051**: they close customers' rooms or send customer messages.

**Why:** "the endpoint exists" ≠ "it runs". Every sweep's doc comment says a CronJob calls it, and none was ever added.
**How to apply:** when a chat-core feature "never happens" on staging, check whether its sweep has a CronJob before debugging logic. A per-workspace sweep needs an all-workspaces fan-out route before a CronJob can call it. Gotcha: `session_summary.Config` seeds `Enabled=true`, and auto-close shares that flag. See [[workflow-tiers-and-decision-inbox]].
