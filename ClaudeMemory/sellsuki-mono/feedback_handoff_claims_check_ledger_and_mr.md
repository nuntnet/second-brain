---
name: feedback_handoff_claims_check_ledger_and_mr
description: "A handoff doc's \"no code yet\" can be wrong — check the card ledger and the repo's open MRs before planning a wave from it"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 28036f73-9028-4b3a-98a8-7ab8e1f2c879
  modified: 2026-11-05T07:15:58.278Z
---

`docs/handoff/ai-chat-2026-11-05.md` said AI-382 had "no chat-core commits yet". In fact the card was built and proved red, and chat-core had MR !193 open (pipeline green). AI-393 was in the same state as !192. Both had since gone into conflict with main.

**Why:** a handoff is a summary written in a rush when the tokens run out. The card ledger, `docs/cards/<CARD>.md`, and the GitLab MR list are the state.
**How to apply:** when picking up from a handoff, check every "still to write" card three ways before telling nunt what is left or briefing a session to build it:
- read its ledger's "Resume here";
- list the open MRs by branch name;
- run `git merge-base --is-ancestor <card branch> <wave branch>`.

Related: [[feedback_check_wave_lists_and_ledger_before_starting_a_card]], [[feedback_list_open_mrs_before_opening_one]].
