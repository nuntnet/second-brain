---
name: ragcore-dual-embedding-paths
description: "rag-core has TWO embedding clients wired at once; one bypasses metering entirely and both are required in staging/prod"
metadata:
  type: project
---

`backend/rag-core` calls embeddings through **two different adapters, both live**:

| | `OpenAICompatibleEmbeddingProvider` | `AIPlatformKitEmbeddingClient` |
|---|---|---|
| file | `src/repository/embedding_repository/openai_compatible.py` | `src/repository/llm_client_repository/ai_platform_kit.py` |
| wired | `cmds/api/wire.py:217` (`_build_embedding`) | `cmds/api/wire.py:362` + `cmds/ingest_worker/wire.py:149` |
| env | `EMBEDDING_BASE_URL` + **own `EMBEDDING_API_KEY`** | `AI_PLATFORM_KIT_BASE_URL` |
| headers | Authorization only — **no workspace/company** | `X-Workspace-Id` + `X-Company-Id` |
| card | none (poc leftover) | AI-39 / AI-41 |

`src/interface/http_server/config/settings.py:216` requires `EMBEDDING_PROVIDER=configured` **and** line 225 requires `AI_PLATFORM_KIT_BASE_URL` when `APP_ENV` is staging/prod ⇒ **both run in production**.

**Why:** the rag-core team did NOT go rogue — the kit adapter's own docstring says it is "the ONE place rag-core's ingest worker is allowed to reach an LLM/embedding capability". The bug is that the poc path was never retired. That path holds its own provider key and reports no scope, so its usage is invisible to metering (violates AI-81 AC5).

**No local model anywhere** — no torch / sentence-transformers / onnx in `pyproject.toml`. Both paths are HTTP to an OpenAI-compatible endpoint.

**How to apply:** AI-133 only needs the *server* side; rag-core's client is written. But AI-133 has **no AC for retiring path A**, so shipping it as written leaves the quota hole open. Also the kit adapter's docstring admits it was written from AI-14's Jira comment rather than `docs/http-contract.md` and asks to be verified — so AI-133's golden contract test and OpenAPI are load-bearing, not nice-to-have. See [[reference-ccs-ai-chat-config-namespace]].
