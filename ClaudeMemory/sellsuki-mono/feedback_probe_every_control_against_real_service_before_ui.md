---
name: feedback-probe-every-control-against-real-service-before-ui
description: "Before building or reporting a settings UI, probe every control × every entity against the real service and build from that table; one happy-path save is not verification"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 70429684-da98-4359-8018-c1ff7471777a
  modified: 2026-11-07T06:32:12.617Z
---

On 2026-11-07 (AI-49 console reset) nunt called a long run of rework "AI slop". Concrete case: the end detail offered toggles (Form, Webhook) on every end; on "ซื้อผ่านลิงก์" chat-core refused with `invalid_event_trigger` because one event trigger may sit on only ONE stage/terminal, and the console mapped that code to a "staff only" toast. Agents had coded against the mock (which has no such rule), and I "verified on local" with a single toggle that happened to succeed (webhook on contact_captured).

**Why:** a mock or a contract in a brief encodes our belief, not the service's rules; one 200 proves one cell. Every reworked screen this session failed the same way — built from assumptions, checked on the happy path, reported as working.

**How to apply:**
- Before briefing UI for a setting, run a probe script against the local real service: every value of every control on every entity (e.g. each trigger on each terminal), record status + error code/message, undo each write. 8 calls took one minute.
- Put that table in the agent brief; require a unit test per cell (allowed / refused-why) and a per-cell table in its report.
- Verify on local by re-running the probe through the UI paths, not one save. Only then say "works".
- When one error code covers several causes, ask the backend to split it rather than guessing in the UI.
Related: [[feedback-verify-as-the-user-sees-it]], [[feedback-mock-mode-ui-review-check-real-adapter-before-calling-defect]].
