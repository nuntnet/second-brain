---
name: reference_bola_jira_project
description: "BOLA tickets live in Jira project BOLA (id 10126), not LR as the repo .claude config claims"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5cc01698-54e5-4bfb-b9e1-d4aba5865591
---

BOLA tickets (e.g. BOLA-157, BOLA-211) are in Jira project **BOLA** — project id `10126`, name "Back office of Line API BOLA", cloudId `50dc7e4a-0539-4dcf-9c33-5481a395a5ab`.

⚠️ The repo config `backend/bola-backend/.claude/rules/jira.md` and `project.md` say Project Key = `LR` (id 10192). That is stale/wrong for these actual tickets — do NOT trust it when looking up real BOLA issues. Use project key `BOLA`.
