---
name: reference_codegraph_watchers_exhaust_system_fds
description: "\"too many open files in system\" / pipe failed / go test infra failures across the machine = codegraph MCP watchers (2 procs held 161k of 184k fds, 2026-10-28) · check kern.num_files, lsof count by pid · user kills them (auto mode denies)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 28036f73-9028-4b3a-98a8-7ab8e1f2c879
  modified: 2026-10-28T15:06:38.027Z
---

2026-10-28 18:50: every command started failing — `OSError: [Errno 23] Too many open files in system`, `pipe failed`, chat-core `go test` failures with `truncate … File exists` / socket errors that looked like flaky tests.

Cause: two `codegraph serve --mcp` processes (one per Claude session) watching the whole monorepo including every `.worktrees/*` checkout held 92,166 + 69,019 fds of `kern.maxfiles` 184,320. The first hit `kern.maxfilesperproc` (92,160).

Diagnose:
```bash
sysctl kern.num_files kern.maxfiles
lsof -n | awk '{print $1,$2}' | sort | uniq -c | sort -rn | head
```

Fix: kill those pids — auto mode denies it ("Interfere With Workloads"), so hand the `kill <pids>` to the user. The MCP reconnects on its own; usage dropped to ~128k after. More worktrees = more watched files, so `/worktree-cleanup` lowers the ceiling too.

Related: `rtk go test` printed "✓ 48 passed" while the raw run had FAIL + panic — read go test through `rtk proxy` ([[feedback_compare_command_outputs_with_rtk_proxy]]).
