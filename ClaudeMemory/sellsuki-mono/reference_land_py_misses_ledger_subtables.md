---
name: reference_land_py_misses_ledger_subtables
description: "scripts/land.py under-reports: MRs listed in a card ledger's extra ### sub-tables are not parsed, so a card with open MRs shows 'merge ครบแล้ว' — confirm with glab before trusting it"
metadata:
  type: reference
---

Seen 2026-09-20 on OC-4581: the ledger had MR tables under `### สไลซ์ B …` and `### AC-06 …`; `land.py` reported "OC-4581 merge ครบแล้ว" while !614 and !650 were open, and bola-frontend !120 showed up only in `--sweep` as an orphan. It reads the main MRs table only.

**How to apply:** when a card has more than one MR round, run `glab api projects/:id/merge_requests/<n>` per MR (state, `detailed_merge_status`, head_pipeline) instead of believing land.py's per-card verdict. Either keep all MRs in one table in the ledger, or fix the parser.
