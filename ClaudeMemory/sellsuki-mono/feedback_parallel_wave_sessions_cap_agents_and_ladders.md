---
name: feedback_parallel_wave_sessions_cap_agents_and_ladders
description: "With several wave sessions running at once, cap each at 2 card agents, one test ladder at a time, console e2e --workers=2 + longer timeout; load hit ~150"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 359b9451-64dc-420f-82ff-ad32178a3b79
  modified: 2026-11-04T11:13:05.521Z
---

When several wave-2 sessions run in parallel on this machine, each one runs at most **two card agents at once**. The others wait their turn. Each session also runs **one test ladder at a time**. The console Playwright e2e runs with `--workers=2` and a longer timeout. A timeout under load is not a result: re-run that spec alone.

**Why:** On 2026-11-04 three wave-2 sessions each ran 5 parallel agents with ladders, and load average reached ~150. The coordinator (AI chat MVP cut) then set this cap.

**How to apply:** Before launching agents in a unit, check whether sibling sessions are active. If they are, queue the agents and resume them via SendMessage as slots free. See [[reference_resume_dead_subagent_by_sendmessage_not_respawn]].
