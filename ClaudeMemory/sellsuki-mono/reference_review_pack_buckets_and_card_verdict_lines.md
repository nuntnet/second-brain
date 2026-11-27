---
name: reference-review-pack-buckets-and-card-verdict-lines
description: "Reading the AI In Review triage artifacts — the 11-24 \"HUMAN\" bucket means \"has any human row\" (agent rows still unverified); verify-sweep files carry a \"Card verdict:\" line to parse instead of AC tables"
metadata:
  node_type: memory
  type: reference
  originSessionId: eda3a5f7-a280-44bf-a2ab-97be594d8200
  modified: 2026-11-27T16:08:18.433Z
---

- In `docs/review-packs/2026-11-24.md`, a card's bucket is **HUMAN if it has any human row**. Its V (agent) rows were never verified, because the 11-25 sweep only took the VERIFY bucket. On 2026-11-27, 57 of the 68 HUMAN cards still had open V rows. So a HUMAN-bucket card is not closable just by moving its human rows to a pack.
- `docs/review-packs/verify-2026-11-25/b*.verdicts.md` has one `## AI-NNN` section per card, and each section has a `**Card verdict: ALL-PASS|HAS-FAIL|INCOMPLETE …**` line. Parse that line. The AC tables use different column layouts across batches, so a script that counts PASS/FAIL cells misreads them. When a card appears in more than one file, `b22` and `b13-b01-recheck` override the earlier files.
- Inside a table, treat `n/a` and `NOTE` cells as non-blocking. A `HUMAN` cell goes to the review pack and does not block closing the card.
- The 2026-11-27 board cleanup split each failing AC into one of 12 bundle cards, labelled `board-cleanup-2026-11-27`. See [[feedback-verifier-closes-cards-review-pack-for-humans]].
