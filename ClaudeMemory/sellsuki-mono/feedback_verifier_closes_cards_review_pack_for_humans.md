---
name: feedback-verifier-closes-cards-review-pack-for-humans
description: "nunt 2026-11-24 — nunt cannot review every AC; verifier verdict counts for agent-observable ACs, only real-person/product-judgment/irreversible-prod ACs go to a one-page review pack; WIP limit 15 In Review"
metadata:
  node_type: memory
  type: feedback
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-24T15:40:42.234Z
---

nunt said (2026-11-24): "AC ทำให้คุณต้องการมนุษย์ตรวจงาน… ผมทำงานไม่ทัน ผมไม่ได้ตรวจเลยซักอย่าง" — ~45 AI cards sat In Review because every AC waited on him. He agreed to the fix, now in `.claude/rules/shipping.md` §1:

- Independent verifier agent (not the author) closes agent-observable ACs (tier A + tier B that only need dev/staging + browser/curl/DB/two companies).
- A person only for: outside channel/real person, product judgment, irreversible prod action.
- Card closes on evidence (on prod + verifier PASS); human-only rows go to `docs/review-packs/<date>.md` — one line each with screenshot/observed value, ~15 min to approve by looking.
- WIP limit: >15 In Review → start no new card.

**Why:** a single human reviewer is the bottleneck; "never grade your own homework" was read as "nunt grades everything".
**How to apply:** when a card's code is merged, run a verifier on its open ACs instead of leaving it In Review; batch the human rows into the review pack; never ask nunt to re-test something a verifier can observe. Related: [[feedback-comment-jira-card-when-loop-closes]].
