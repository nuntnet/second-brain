---
name: project_ai146_onboarding_wizard_not_started
description: The AI chat admin has no onboarding flow at all; AI-146 is the To Do card for it and the plan doc specifies a <30-minute template wizard
metadata:
  type: project
---

The AI chat admin console has **no onboarding, no setup wizard, and no readiness
signal**. Searching `onboarding|setup|wizard|checklist|getting started` across
`frontend/ai-chat-admin-frontend/apps/admin` returns nothing, and there is no
workspace-level `not_configured` / `is_ready` concept in `backend/sellsuki-chat-core`.

An admin opening a fresh workspace lands on an empty Inbox with nothing telling them
what to configure. The working order (company → workspace → members → connect FB Page
→ persona → KB → playbook → tier-0 → tier-1 → guardrails → SLA → eval → kill switch,
plus AI-115's conversation goal which has no UI) exists **only in the code**, written
down nowhere the admin can see.

This is specified product, not an oversight of the spec:
- `docs/ai-chat-assistant-platform-plan.md` §5.11 line ~449: *"UX onboarding แบบ
  Trello: wizard เลือก template → pre-provision ครบ → ต่อ channel → ทดสอบ → เชิญทีม
  (<30 นาที)"*
- Same section defines **Solution Template** = `fact schema + intent pack + persona +
  goals + case statuses/rules + dashboard layout + label pack + sample KB`. Nothing
  named "template" exists in the code.
- **AI-146** `[E12][Platform] Solution Template catalog + onboarding wizard` — **To Do**,
  under epic **AI-131** (E12, To Do). Sibling **AI-152** HR template also To Do.
- `AI-111` (W2 delivery) separately owes an "onboarding runbook" document — To Do.

Do not read the 2026-08 gap sweep note as "E12 is finished" — AI-131/146/152 are all
To Do. See [[project_ai_backlog_gap_sweep_202608]].

**Why:** a read-only setup checklist can ship against APIs that already exist and does
not need the template engine, so it is separable from AI-146's write side. Conflating
the two makes a shippable UX fix look like a platform epic.

**How to apply:** when asked "what does an admin do first", say plainly that the
product does not answer that question yet, and that AI-146 is the card. Relates to
[[project_ai115_conversation_goal_backend_only]] and
[[feedback_selfexplaining_ux]].
