---
name: report-wrong-cards-dont-edit
description: "When a Jira card is wrong or self-contradictory, report it and let the user decide — and prefer fixing code over opening new cards for small defects"
metadata:
  type: feedback
---

Two rulings from 2026-08-31, both after I proposed the wrong thing.

**1. Don't silently correct a card — surface the contradiction.** The user's
words: "ถ้าพบว่าการ์ดขัดแย้งหรือผิด แจ้งผม จะได้เคาะให้ว่าแก้มั้ย". Commenting
with evidence is fine and expected; rewriting the card's scope or AC is theirs
to decide.

**Why:** in one session I found six cards materially wrong (AI-103 blamed the
wrong blocker, AI-149 and AI-193 declared blocked while their backends existed,
AI-129 carried an AC no backend enforces, AI-150 omitted the display-name
problem entirely, plus one of my own earlier comments). A card's wrongness is
usually a signal about the product decision behind it, not a typo to patch.

**2. Don't open a Jira card for a small code defect — fix it.** I proposed
cards for two flaky tests and a coverage gate; the user pushed back: "ต้องเปิด
การ์ดหรอ เพราะการ์ดเขียนผิด?" Correct. A card describing a 20-minute fix
becomes another stale card — see [[oc-epic-backlog-triage]] (OC board: ~127
non-Done epics, 26 real).

**How to apply:** sort the finding first.
- broken code, small → fix it now, in a branch, with a mutation check
- a decision nobody has taken (a threshold, an ownership question) → surface it
  in one line and ask; it is not a task and a card will just sit
- real multi-step product work → then a card earns its place

Related: [[ground-claims-file-line]], [[verify-absence-claims]],
[[decisive-deep-execution]].
