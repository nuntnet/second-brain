---
name: reference_zsh_colon_modifier_eats_git_refspec
description: "In zsh, `$VAR:refs/...` and `$VAR:src/...` apply history modifiers (:r, :s) to the variable — git refspecs and `<rev>:<path>` silently break; always write `${VAR}:...`"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4a70053d-20ab-45c5-8ae9-5097398e8ddc
  modified: 2026-11-02T20:32:01.640Z
---

Seen twice on 2026-11-02/03 (AI-374): `git push origin $C:refs/heads/x` became `<sha>efs/heads/x` ("src refspec … does not
match any"), and `git show $B:src/use_case/...` became `…oup.go` (unknown revision). zsh treats `:r` / `:s` after a
bare `$VAR` as modifiers.

**How to apply:** in Bash-tool commands (zsh) brace every variable followed by a colon: `"${C}:refs/heads/<b>"`,
`"${B}:path"`. Related: [[reference_backticks_in_double_quoted_python_c_execute]] (same session also hit unquoted
heredoc backticks executing — use `<<'EOF'` and pass values via env).
