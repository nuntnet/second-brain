---
name: reference-codegraph-context-needs-projectpath
description: codegraph_context answers from the wrong service at monorepo scope; always pass projectPath to a repo-local index
metadata: 
  node_type: memory
  type: reference
  originSessionId: cac73755-22e9-4905-b12d-0973d98763b2
  modified: 2026-08-29T16:30:44.494Z
---

In sellsuki_mono, `codegraph_context` — the tool the MCP server advertises as
PRIMARY — is wrong more often than right at monorepo scope, and it fails
**silently**: well-formatted, confident results from an unrelated service.
Measured 2026-08-29: "bola-backend LINE webhook reply token" → order-management's
`Handle()`; "space-go PIS grpc client" → Python classes in
`testing/oc2plus-line-crm-automate-testing`; "chat-core assign operator" →
generated TS type aliases, missing `src/use_case/operator_assignment.go` that
`codegraph_search` returns as hit #1. Naming the repo in the task text does not
help — the ranker has no path scope, so one generic token (`Handle`, `Server`,
`Repository`) matches across 40+ repos.

Every repo with ≥20 source files now carries its own `.codegraph/` index. Pass an
absolute `projectPath` to it and the same questions answer correctly. The
"⚠ results come from a different git worktree" banner on scoped calls is a false
alarm (it compares to the MCP server's root) — ignore it.

`codegraph_search` / `callers` / `callees` / `impact` / `trace` are accurate at
monorepo scope, 0.2–0.5s. Only `context` needs scoping.

**Why:** the repo has ~26 backend + 13 frontend submodules; FTS ranking across all
of them has no way to prefer the service you meant, and 53% of the monorepo graph
is `import` nodes plus generated code (`*.pb.go`, `*.gen.go`, `mock_*`) that
outranks real symbols.

**How to apply:** `codegraph_search` first to find the repo, then
`codegraph_context` with `projectPath`. Refresh repo-local indexes with
`make codegraph-sync` (they have no file watcher — only the monorepo-wide index
auto-syncs). Never `rm -rf` a `.codegraph/` while a session is live: the running
MCP server keeps serving the deleted inode, so the CLI is right and the tool is
stale with no error. Use `scripts/codegraph-sync.sh --rebuild` instead. Full
detail in `.claude/rules/codegraph.md`. Related: [[reference-submodules-are-shallow-clones]],
[[feedback-head-on-grep-is-sampling-not-verification]].
