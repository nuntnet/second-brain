---
name: reference_backticks_in_double_quoted_python_c_execute
description: "Markdown backticks inside a double-quoted `python3 -c \"...\"` (or any \"...\" shell string) are command substitution — they RUN; put MR/Jira text in a file, never inline"
metadata:
  node_type: memory
  type: reference
  originSessionId: 77b80035-3131-4ced-9fb5-6ef6750029a3
  modified: 2026-11-02T22:08:44.456Z
---

On 2026-11-02 (AI-358) an MR-description runbook was built inline in `python3 -c "..."`. Its markdown code spans
(`` `MIGRATE_HEADLESS=true go run ./cmd/migration/` ``, `` `SELECT count(*) ...` ``) were executed by zsh as command
substitution inside the double quotes. The migration tool actually started in the main chat-core checkout and only
stopped because `DATABASE_CHAT_CORE_POSTGRESQL_URI` was unset.

**How to apply:** any text containing backticks (MR descriptions, Jira comments, runbooks) goes into a file with the
Write tool, then a small script reads the file (`--input body.json` for glab). Never interpolate markdown into a
double-quoted shell string; a quoted heredoc (`<<'EOF'`) is safe, `"..."` is not. **An UNquoted heredoc (`<<EOF`) expands
backticks too.** On 2026-11-03 the coordinator wrote a wave-file row through `python3 - <<EOF` (unquoted, so it could use
`$TMPDIR`). Every code span ran as a command and vanished from the committed row. They were harmless that time (not found /
permission denied), but a span like `` `go run ./cmd/migration` `` would have run. If you need a shell variable, pass it as
an argv (`python3 - "$TMPDIR/x" <<'EOF'`) and keep the heredoc quoted. Related: [[reference_auto_mode_blocks_glab_mr_create]].
