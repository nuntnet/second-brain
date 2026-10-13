---
name: workflow-tiers-and-decision-inbox
description: "2026-10-13 user's workflow pain (hard-to-test ACs block MRs, parallel agents pile up, questions sink in chat) and the two fixes adopted - AC tiers A/B/C and the Decision Inbox; not yet eval'd or committed"
metadata:
  node_type: memory
  type: project
  originSessionId: c18893a0-7a71-4ff1-818b-823205c08f58
  modified: 2026-10-13T11:25:45.491Z
---

On 2026-10-13 the user (PO + dev of the sellsuki workflow) described the root problem: cards carry many ACs, some hard for an agent to test (OTP, file upload/read, screen rendering, and "other kinds too"), so MRs never open, new work stacks on unmerged work, parallel agents become spaghetti, and questions for the user scroll away in chat. The user **insists parallel agents stay**; the working method must change, not the parallelism.

Adopted (user approved, "ลองทำเลย"):
- **AC tiers** A (agent-observable) / B (needs dev + person or outside channel) / C (staging/prod only). User explicitly OK'd opening the MR into `develop` once tier A passes; B blocks `main`, C blocks release. Files: `.claude/knowledge/ac-tiers.md`, `shipping.md` §2, `feature.md`, `docs/cards/TEMPLATE.md`, `scripts/ledger-check.py` (status `blocked-on-dev`).
- **Decision Inbox**: questions go to `docs/decisions/INBOX.md` via `scripts/inbox.py` with a default and "if wrong" cost; sessions never wait; `inbox.py check <CARD>` gates main. Replaced the old "one blocking question per loop" (`shipping.md` §6).

**Why:** the card-done gate and the chat-question habit were the bottlenecks, not code quality. The user values the multi-env testing (local → dev → staging) and does not want it weakened.

**How to apply:** classify every AC's tier at refinement (human confirms); never tier down your own ACs to dodge them; write questions to the inbox, not chat. Still open as of 2026-10-13: `/eval-skill` not run on either change, nothing committed (feature.md/TEMPLATE.md/shipping.md also carry other sessions' uncommitted edits, use `git add -p`), no hook prints `inbox.py count` at session start (settings.json is outside sandbox write access), old ledgers' ~270 unanswered Open-questions rows not imported, and PO card writing (`po`, `/write-jira-card`) does not yet assign tiers. See [[feedback-comment-jira-card-when-loop-closes]].
