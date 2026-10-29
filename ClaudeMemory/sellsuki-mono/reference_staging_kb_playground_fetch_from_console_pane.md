---
name: reference_staging_kb_playground_fetch_from_console_pane
description: "Read-only repro of the staging KB playground: from the logged-in ai-chat-admin.staging-th pane, fetch POST https://chat-core.staging-th.sellsuki.com/v1/workspaces/<ws>/kb/playground with credentials:'include' + X-Company-Id; response carries citations[].score and retrieval_mode. Playground citation score is a RETRIEVAL score, not the FAQ-match score (AI-344)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 5854587f-1e69-4061-92b3-46d4bb104b14
  modified: 2026-10-29T06:00:37.174Z
---

Used 2026-10-29 on AI-344 (pilot ws 5266e73a-c55d-434b-835c-17ce57507a08, company f2bf928c-7cd0-4059-9326-82fddeba4933).

- In the built-in browser pane already logged in to https://ai-chat-admin.staging-th.sellsuki.com, `javascript_tool`:
  `fetch('https://chat-core.staging-th.sellsuki.com/v1/workspaces/<ws>/kb/playground',{method:'POST',credentials:'include',headers:{'Content-Type':'application/json','X-Company-Id':'<company>'},body:JSON.stringify({question:'…'})})`
  It is a preview (stores nothing), so it is not a remote write. The console UI drops `score`/`retrieval_mode`; only this raw call shows them.
- Before the AI-344 fix (chat-core ≤ ef2211a), the playground never ran FAQ-first: an exact FAQ question shows a document score (~0.74), and that scale is NOT the FAQ threshold's. After the fix: `retrieval_mode: "faq"`, score 1.0 for exact text.
- A 502 `playground_retrieval_failed` usually means rag-core-python cannot reach ai-agent for embeddings (`kubectl logs deploy/rag-core-python` shows `ai_platform_kit.embed ConnectError`); check `kubectl get endpoints sellsuki-ai-agent-svc`.
- Related: [[reference_staging_ai_chat_console_drive_recipe]], [[reference_staging_chat_core_replay_recipe_in_pod_wget]]
