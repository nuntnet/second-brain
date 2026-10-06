---
name: reference_google_drive_mcp_shared_files_by_id
description: "Google Drive MCP in this workspace — search_files returns {} for files other people shared (incl. parentId queries); open them by file ID taken from RAG citation source_url; Drive's PDF text export strips Thai vowels so never use it as an extractor"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-06T11:37:22.786Z
---

- `search_files` (title/fullText/parentId/sharedWithMe) returned `{}` for every file in the FWD KB folder
  `1jGaGm3ipoShashcVg_4mTjST0HMsZ5s7` ("ข้อมูลประกันลูกค้า", owner amonpan.ler@; most files owned by
  m.threewattana@gmail.com). `get_file_metadata` / `read_file_content` by ID work fine.
- Fastest way to get the IDs: pull `source_url` from RAG evidence/citations
  (`qa/kb100-2026-10-05/*.jsonl`) and regex `/d/<id>`.
- `read_file_content` on a Thai PDF returns Google's text layer with vowels/tone marks dropped
  ("เอฟดบบลวด พรเชยส") — useless as evidence; keep using the worker's poppler/pdfplumber path. Google Sheets
  come back as per-sheet markdown tables with ranges, which is good enough to verify empty rows.
- The FAQ sheet "AI Insurance Assistance Data" really is empty for 25/15, 30/15, Precious Care at the source
  (checked 2026-10-06); only 90/20 and CI50 tabs have Q&A. Its CI50 tab says 16–64 and 90/20 says to 70,
  conflicting with the brochures (16–65, 65).
