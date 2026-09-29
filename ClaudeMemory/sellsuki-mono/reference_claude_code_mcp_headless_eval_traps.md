---
name: reference-claude-code-mcp-headless-eval-traps
description: "Running tool-selection evals through `claude -p` against the user's OAuth MCP server — two traps that invalidated OC-4625 B9's first run (credential cleared mid-run; whoami-first by design for multi-company users)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4341156f-9031-4920-83b6-fd7bcbf45cd4
  modified: 2026-09-29T04:23:50.879Z
---

Found 2026-09-29 running OC-4625 AC-B9 (46 questions) with `claude -p … --output-format stream-json --allowedTools mcp__oc2plus-backoffice` against the user's user-scope MCP server.

1. **Do not run a second config with the same MCP server name while an eval is running.** A probe with `--mcp-config pinned.json --strict-mcp-config` that reused the name `oc2plus-backoffice` (to pin a company with a header) came up `needs-auth`, and from then on every session of the main run did too (`first=None` for 23/46). Oathkeeper showed no denials and Hydra no failed token calls, so the gateway was fine — the stored credential for that name was what broke. The user had to Authenticate again in `/mcp`. Also run the questions **sequentially** (4 parallel sessions share one refresh token).
2. **A multi-company user makes the model call `whoami` first** (21/23), because the server instructions forbid guessing a company_id. That is correct behaviour, so "first tool called" cannot be the metric for such a user — score the first tool *after* whoami (and report the strict score too).

Also: headless sessions first call `ToolSearch` to load deferred MCP tools; filter tool_use names by the `mcp__<server>__` prefix. Pass `< /dev/null` or `claude -p` waits 3s for stdin.

Related: [[project-oc2plus-mcp-assistant-idea]]
