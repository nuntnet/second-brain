---
name: feedback-open-mr-per-wave-or-batch-not-per-card
description: "nunt 2026-11-10: do not open one MR per card; open MRs per wave or batch. Said while I was preparing a single-card MR for BOLA-383 (PAT Phase 5 set)"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 300c6962-12fa-4740-9aa3-3eb3ab997d45
  modified: 2026-11-13T03:25:12.188Z
---

On 2026-11-10, after I reported that I would "prepare the MR for BOLA-383" (one card of the PAT Phase 5 set, epic BOLA-382), nunt wrote: "ไม่เปิด mr ทีละ card ให้เปิดทีละ wave หรือ batch".

**Why:** not stated in that message. The existing wave exception in `.claude/rules/shipping.md` §2 (AI chat MVP, 2026-10-30) gives the reasons nunt used before: each MR pipeline costs about an hour, runners fail often, and one bot review per card is paid many times. That those reasons apply here too is my reading, not nunt's words.

**How to apply:**
- For the PAT Phase 5 BOLA cards (BOLA-383..388): each card gets its own branch cut from `main`, pushed, **no MR**; tier-A verifier verdicts on local as usual; at the end of a wave cut `feature/<wave>` from `main` per repo, merge card branches with `--no-ff`, open one MR per repo with a card → AC → verdict table. Wave list: `docs/cards/waves/PAT5-wave1.md`.
- Do not say "I will open the MR for <card>" in a report; say which wave the card belongs to and what the wave is waiting on.
- **Scope confirmed 2026-11-13:** nunt: "เพราะเราเลิกทำ MR ต่อ card แล้วเรารวมเป็น wave". This is now the default, not just for PAT5. (`shipping.md` §2 still describes one MR per card as the rule outside the AI chat wave; the text is behind the practice.)
- **Consequence for sweeps:** a card can be on `main` with no MR that names it. To decide whether a card is built, check git ancestry (`git merge-base --is-ancestor <card-branch-or-commit> origin/main`, commit messages carrying the card id), never MR titles. On 2026-11-13 I nearly moved 12 In Review cards back to To Do/In Progress because no MR named them; all 12 were on main via waves.

Related: [[feedback_check_wave_lists_and_ledger_before_starting_a_card]], [[feedback_wave_integration_full_suite_and_tsc_on_merged_tree]], [[feedback_parallel_wave_sessions_cap_agents_and_ladders]].
