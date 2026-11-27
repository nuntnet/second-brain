---
name: feedback-give-user-one-line-commands-not-heredocs
description: Commands nunt must run himself should be ONE short line (a script file in the repo), not a multi-line heredoc or a JSON snippet to paste into zsh
metadata:
  node_type: memory
  type: feedback
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-27T17:11:04.435Z
---

2026-11-27: I gave nunt a JSON snippet "to add to settings.json" — he pasted it into zsh (`parse error near ]`). Then a multi-line `python3 - <<'EOF'` command — the heredoc broke on paste and nothing was written (settings.json identical to the `.bak`). A one-line `python3 scripts/hooks/install.py` worked first time.

**Why:** pasting multi-line blocks into a terminal is fragile (heredocs, quoting, the app's Run button runs one line), and he cannot tell from the output that it silently did nothing.

**How to apply:** when nunt has to run something himself (sandbox-protected files, Vault, prod), put the logic in a small idempotent script in the repo (with `--check`, a backup, and clear "added/installed" output) and hand him one line. Afterwards verify the effect yourself (read the file / `--check`) instead of trusting that it ran.
