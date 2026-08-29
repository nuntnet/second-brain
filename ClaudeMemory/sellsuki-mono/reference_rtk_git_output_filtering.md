---
name: rtk-output-can-drop-lines
description: The rtk token-saving hook can silently drop lines from command output — verify with grep before concluding something is missing
metadata: 
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-27T17:47:16.019Z
---

The global `rtk` hook (see ~/.claude/RTK.md) rewrites shell commands to a token-optimizing proxy. It does not only compress — it can **silently omit lines**, so absence in the output is not evidence of absence in the file.

Observed cases:
- `cat package.json` came back missing a script line, making a CI gate (`depcheck`) look nonexistent — nearly reported as a CRITICAL "broken gate" until re-checked with `grep`.
- `git` output has come back compressed or empty where the exact text mattered (e.g. a blank commit body).

- 2026-08-11: `ls -1 <dir>` and `find … -name …` inside a submodule came back **completely empty** (dir was
  fully populated). Same commands via `/bin/ls` / `/usr/bin/find` returned everything. So it is not only
  compression of long output — plain listings can come back as nothing at all.

- 2026-08-28: `tail -40 <file>` was rewritten into a **different command entirely** — it errored with
  `/usr/bin/read: line 4: read: -4: invalid option` plus bash's `read` usage text. The hook mapped `tail`
  onto `read`, so the flag became garbage. Nothing was read at all. Use the Read tool, or `/usr/bin/tail`,
  when tailing a file matters.

- 2026-08-29: the worst case so far — `diff old new` reported **"✅ Files are identical"** for two files that
  genuinely differed (the edit had landed). A rewritten `diff` that lies in the *affirmative* is not caught by
  "re-grep when something looks missing": it looks like successful verification. `/usr/bin/diff` showed the
  change immediately. Same session: a `cat > file <<'EOF'` heredoc failed with "no such file or directory"
  (write never happened — use python or the Write tool), and `npx playwright test --reporter=json > out.json`
  wrote the hook's own 16-line summary into the file instead of the JSON, until run as
  `./node_modules/.bin/playwright`.

**Rule:** never let a verification step run through the hook. When a conclusion depends on what a command printed — a line missing, a diff empty, a file written, a test report parsed — invoke the binary by full path (`/usr/bin/diff`, `/bin/ls`, `./node_modules/.bin/<tool>`) or use the built-in Read/Write/Edit tools. Re-grep when something looks absent, and distrust a *clean* result just as much: the hook has reported "files are identical" for files that were not.
