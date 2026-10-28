---
name: reference_codegraph_mcp_watchers_exhaust_macos_file_table
description: "Two codegraph MCP watcher node processes held ~92k and ~69k fds on 2026-10-28 and exhausted the macOS system-wide file table (ENFILE, 'too many open files in system') so every Bash call failed; find them with lsof -n -c node, killing another session's process needs nunt's explicit OK in auto mode"
metadata:
  type: reference
---

On 2026-10-28 ~18:40 ICT every Bash call in this workspace failed with ENFILE ("too many open files in system"). The holders were two `@colbymchenry/codegraph` MCP watcher processes started by other Claude sessions (PID 40460 ~92k fds, PID 47251 ~69k fds), each watching `.codegraph/codegraph.db-shm` across the submodules. `ps` is blocked in the sandbox; `lsof -n -c node | awk '{print $2}' | sort | uniq -c | sort -rn | head` ranks the fd holders and `lsof -p <pid>` names the executable.

**Why:** the system file table is shared by every session on the machine, so one runaway watcher takes every other session down with it, and the symptom (ENFILE on `git`, `cat`, anything) looks like a broken sandbox rather than a hungry process.

**How to apply:**
- On ENFILE, rank fd holders with lsof before touching the sandbox; look for codegraph watchers first.
- The auto-mode classifier denies `kill` of another session's process ("Interfere With Workloads"); ask nunt, who authorized it in one line ("อนุญาตให้ kill"). By the time they answered the watchers had exited on their own, so a short wait is also an option.
- After the table recovers, this session's own codegraph/outline MCP connections may stay timed out until restart; that does not block git or file work.

Related: [[reference_codegraph_context_needs_projectpath]], [[reference_harness_classifier_blocks_secrets_and_mutations]].
