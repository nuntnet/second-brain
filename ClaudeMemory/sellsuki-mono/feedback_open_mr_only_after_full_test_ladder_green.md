---
name: feedback_open_mr_only_after_full_test_ladder_green
description: "nunt 2026-11-02: open an MR only after the WHOLE test ladder is green — unit, integration on a real DB with -p 1, and the ledger's AC→test table with every test proved red; producer (kit) MR before consumer"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 3c6e4362-8fae-41c1-a18f-6e53fb5cee11
  modified: 2026-11-02T04:31:30.897Z
---

nunt, relayed on AI-382 (2026-11-02): "ทำต่อ ให้เสร็จ test ครบแล้วค่อยเปิด mr".

**Why:** an MR opened on a partly-run ladder burns a slow CI run and a review on work that is not proven; on the AI chat repos CI takes ~1h and often fails on runners.

**How to apply:** before creating any MR — (1) unit tests, (2) the repo's full suite against the real local Postgres with `-p 1` (chat-core), (3) the card ledger has the AC → test → result table and every test was proved red (named `--- FAIL`, not a build error). Only then open MRs, producer first (kit → chat-core → console), with the order written in each MR. Related: [[feedback_tests_derive_from_the_card_not_the_code]], [[feedback_prove_red_script_never_pipe_into_head]].
