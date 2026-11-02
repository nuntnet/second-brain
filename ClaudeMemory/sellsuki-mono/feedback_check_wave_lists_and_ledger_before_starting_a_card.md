---
name: feedback_check_wave_lists_and_ledger_before_starting_a_card
description: "Before starting or briefing any AI chat card, grep docs/cards/waves/*.md and look for docs/cards/<CARD>.md; a wave card gets no MR of its own and may already be built"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 8b69c115-30aa-4ff4-b9c5-86fafae17c77
  modified: 2026-11-02T10:00:28.822Z
---

On 2026-11-02 I started a session to "implement" AI-355, and an earlier one for BOLA-381 with "open the MR into develop". Both cards are units of `docs/cards/waves/MVP-wave2.md`.
- AI-355 had already been built and verified on 2026-10-30. Its branches were pushed with no MR and `docs/cards/AI-355.md` existed, so the new session started redoing finished work.
- BOLA-381 opened its own MR (!275), which the wave rule in `shipping.md` §2 does not allow.

I had checked only `MVP-wave1.md`, and only for other cards.

**Why:** the wave rule works only if every session knows which cards are in a wave. A card's Jira status ("To Do") says nothing about it: wave cards stay To Do with the code pushed.

**How to apply:** before writing any brief, run `grep -l <CARD> docs/cards/waves/*.md` and `ls docs/cards/<CARD>.md`.
- If the card is in a wave: the brief says "push the branch, no MR, verify on local, record it in the wave file".
- If a ledger exists: the brief says "resume from its Resume here" (`/resume`), not "implement".

Related: [[feedback_tests_derive_from_the_card_not_the_code]], [[feedback_open_mr_only_after_full_test_ladder_green]].
