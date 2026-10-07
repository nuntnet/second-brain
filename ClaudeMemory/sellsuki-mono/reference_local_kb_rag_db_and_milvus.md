---
name: reference_local_kb_rag_db_and_milvus
description: "Where the local rag-core KB data actually lives — Postgres kb_rag_local (docker sellsuki-rag-postgres, no psql on host), table canonical_knowledge_document, Milvus kb_rag_oauth_1536_v1 — and the stale look-alike DB to avoid"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-06T11:37:22.729Z
---

The running local rag-core (`pgrep -f cmds.api`, port 8001) uses
`POSTGRES_DSN=…@127.0.0.1:55432/kb_rag_local` and `MILVUS_URI=http://127.0.0.1:19531`
(`MILVUS_COLLECTION_KNOWLEDGE=kb_rag_oauth_1536_v1`, `EMBEDDING_MODEL=text-embedding-3-small` 1536,
`LLM_*_MODEL=openai/gpt-4o-mini`). Read the env off the process, not `.env` — the file has those lines
commented out.

- No `psql` on the host: `docker exec sellsuki-rag-postgres psql -U postgres -d kb_rag_local -At -c "…"`.
  In zsh, don't put the command in a variable with flags; use a shell function.
- Tables: `canonical_knowledge_document` (markdown, sections, metadata.source_extraction.text_engine),
  `knowledge_chunk` (metadata only — chunk TEXT lives in Milvus, not Postgres), `structured_knowledge_table`,
  `knowledge_wiki_page`, `knowledge_ingest_job`.
- `company_rag` on the same server is an OLD schema (no canonical table). Don't diagnose against it.
- Test workspace for the FWD KB: company `60daa2a8-65cc-4259-b2b1-8f96fb504ff3`, workspace
  `5fcfb2df-afc9-4954-8d70-33fdb7ddd674`; 12 active docs, 823 chunks, 378 of them from one formula xlsx.
- chat-core workspace config is in `sellsuki_mono-postgres-1` db `chat_core`, table `workspaces`
  (`model` column, default gpt-4o-mini).
- PDF/XLSX extraction for Drive imports runs in `data-pipeline/rag_knowledge_base_data_pipline/src/knowledge_connector_worker/`
  (pdf_layout.py, pdf_spans.py, xlsx_layout.py), not in rag-core's `standard.py`.

**Milvus will not start after an unclean Docker stop (2026-10-07).** `sellsuki-kb-rag-milvus-1` (milvusdb/milvus
v2.6.13 standalone, `ETCD_USE_EMBED=true`, volume `sellsuki_kb_rag_milvus_26`, compose `docker-compose.kb-rag.yml`
at the monorepo root) exits 134 within 3-6 s every time with `panic: etcdserver: leader changed`: Milvus reads
`by-dev/meta/session/id` before the embedded etcd has elected itself, and the linearizable read fails when the
leader appears. Retrying (4x), waiting for low load, and a faster election config (`heartbeat-interval: 20`,
`election-timeout: 200` via a compose override) all failed identically. etcd data is small (125 MB, db 17 MB), the
disk is 94% full. Non-destructive fixes exhausted; the remaining option is wiping the volume
(`docker volume rm sellsuki_kb_rag_milvus_26`) and re-ingesting the workspace's 12 documents — ask first.
