---
name: reference-rag-core-mcp-prior-art
description: A working Sellsuki MCP server already exists — the Go rag-core in GitLab sellsuki-rag/rag-core (not in the workspace), built by kimzey (Pan Nitiyothin) on PAT-2691. It is the template for any new MCP.
metadata:
  type: reference
---
Found 2026-09-27.

**Where it lives**
- The deployed rag-core is **GitLab `sellsuki/sellsuki/sellsuki-rag/rag-core`** (Go).
- The workspace's `backend/rag-core` is a different project, `sellsuki-rag/poc/rag-core` (Python), and has no MCP code. That is why a search inside the workspace finds nothing.
- Staging runs image `rag-core:a336e46d` in ns `sellsuki` with env `MCP_ENABLED`, `MCP_PUBLIC_URL`, `MCP_RESOURCE_PATH` and `ROLE_PERMISSION_GRPC_ADDR`.
- The sibling repo `sellsuki-rag/sellsuki-mcp-connectors` holds the client-side plugin and config (Claude Code, Codex, Cursor, Gemini) plus a skill.
- Server code: `route/mcp` sits inside the service itself (in-process), built on the official `modelcontextprotocol/go-sdk` v1.7.0 with stateless Streamable HTTP. A server is built **per request**, and the company enum is placed in the schema. It serves PRM at `/.well-known/oauth-protected-resource`.
- The Verifier trusts X-User-Id/X-Client-Id from Oathkeeper. There is a single public client `sellsuki-rag-mcp` and **no DCR**, so most clients need the client id typed in by hand.
- Company ladder: path `/mcp/:company_id` → header `Mcp-Param-CompanyId` → argument → selected company (central-config `sellsuki-global-user`, the same value the web UI uses) → sole membership → `company_required`.
- The gate is **membership only** (rps ListAssignedRoles), not a per-action permission, and it is cached for up to 300s. All tools are read-only. The only write is `switch_company`, which changes the selected company in the web UI too.

**Who built it:** kimzey = Pan (Kim) Nitiyothin (pan.nit@sellsuki.com), 2026-09-10 to 09-18, card PAT-2691.

**Patterns it already solved**
- `switch_company` tool, with a per-caller `company_id` enum drawn from rps membership.
- Retrieval gated on company membership.
- A tenant-isolation test.
- `OAUTH_REQUIRED_SCOPES` was deleted. It advertises `offline` so Oathkeeper's inherited global `required_scope` (user.read + offline) passes.

**Supporting changes in kratos-ui-go**
- `6400fa4`: 60KB proxy buffer, because Gemini's state parameter overflowed the default.
- `e06c69f`: RFC 9207 `iss` so Codex/ChatGPT get a fixed callback. Hydra v2.2.0 cannot do this itself.
- Claude worked without either change.

**rps:** `62324ad` seeded the `rag.*` catalog.

Related: [[reference-oathkeeper-hydra-cluster-state]] · [[project-oc2plus-mcp-assistant-idea]]

**From Pan, 2026-09-28:**
- The rag-core MCP knowledge base is **not yet separated by company**. Anyone who logs in and asks gets Sellsuki's own knowledge, because Sellsuki has no company_id field yet. The team data work will add a company_id filter. Until then customers must not be given the connector.
- The connectors repo can hold several products' plugins; users install per plugin. It is not public yet (planned for GitHub later).
