---
name: feedback-before-closing-card-count-jira-acs-against-verified-rows
description: "2026-11-25 verify sweep — closed AI-92/96/101 on DoD rows only because the AC parser missed Jira's escaped `\\[ \\]` bullets; AI-246 closed while its AC-03 test was never on main. Before Done: count Jira AC bullets vs verified rows, and check each AC's commit is an ancestor of the prod tag"
metadata:
  node_type: memory
  type: feedback
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-25T07:24:02.013Z
---

On 2026-11-25 I moved AI-92, AI-96 and AI-101 to Done after the verifier passed every row in its batch file. The batch file only held the DoD rows: the Jira AC section writes bullets as `* \[ \] …` (markdown-escaped), and the classifier regex only matched `- [ ]` / `**ACn**`. Twelve cards were affected (AI-92 96 99 101 106 149 198 236 237 238 240 242). Separately, AI-246 was closed on a staging PASS of AC-04 while AC-03's test (commit `4a12007`) sat only on codex branches, never on main or the prod tag. I reopened all four with a correction comment.

**Why:** a verifier PASS covers only the rows it was given; "all rows PASS" is a claim about the input file, not the card.
**How to apply:** before moving any card to Done, (1) count the card's AC bullets in Jira (match `\[ \]`, `[ ]`, `**AC`, `AC-0n`) and confirm each one has a verdict; (2) for any AC whose evidence is a commit/test, `git merge-base --is-ancestor <sha> <prod tag sha>`. Related: [[feedback-verifier-closes-cards-review-pack-for-humans]].
