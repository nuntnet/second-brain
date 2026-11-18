---
name: jira-bulk-thai-edits-drift-use-ascii-escaped-json
description: "Long Thai Jira descriptions re-typed by an agent drift silently (words swapped/added, even in the 'original' section); emit the file as ASCII-escaped JSON and paste that, then diff Jira against the file"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4118eddb-7d1a-4a35-96ee-abb152cd4c4d
  modified: 2026-11-18T14:34:19.210Z
---

2026-11-18, sharpen of AI Sprint 25 (14 cards): a sonnet subagent posted 12 prepared descriptions with `editJiraIssue`. AC item counts all matched, but a later diff of Jira vs the source files found 5 of 12 drifted: "ทับกับ"→"ทับซ้อนกับ", "ข้อต่อไป"→"ข้อถัดไป", "ทิศ"→"ทิศทาง", and one original-AC line gained words copied from another paragraph. The agent re-typed Thai from memory of the file, it did not copy bytes. Item-count checks cannot catch this.

**What worked:** `python3 -c "import json;print(json.dumps({'description':open(F).read()}))"` (default `ensure_ascii=True`) and paste that `\uXXXX` JSON straight into `fields`. Zero drift on every card sent that way.

**How to apply:** for any bulk Jira text write, (1) send from ASCII-escaped JSON, never from re-typed text; (2) after the batch, fetch all descriptions in one JQL (`fields:["description"]`, markdown) and diff each against its source file with `-`/`*` bullets and `\_` escapes normalised; resend only the cards that differ. Don't delegate the paste to a subagent unless it uses the same JSON method.

Related: [[reference_jira_editissue_adf_breakage]], [[reference_jira_mcp_editissue_fields_rejected_as_string]].
