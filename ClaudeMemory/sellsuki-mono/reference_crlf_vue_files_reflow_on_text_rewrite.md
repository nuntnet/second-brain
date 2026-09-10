---
name: reference_crlf_vue_files_reflow_on_text_rewrite
description: "Some source files in the OC2Plus backoffice FE are CRLF (src/layouts/Default.vue); rewriting one with a naive text-mode script flips every line to LF and turns a 2-line change into a whole-file diff"
metadata:
  type: reference
---

`frontend/oc2plus-linecrm-frontend-backoffice/src/layouts/Default.vue` is stored with CRLF
line endings (and mixes tabs into an otherwise space-indented file). Editing it with
`python3 io.open(..., encoding='utf-8')` read + write, or any text-mode rewrite, silently
translates CRLF → LF on write: the file content is right, but `git diff` shows the entire
file removed and re-added, which buries the real change and makes review/merge conflicts far
worse.

**How to apply:** when a scripted edit produces a whole-file diff, don't commit it — `git
checkout --` the file and redo the edit in **binary mode** (`open(p,'rb')` / `replace` on
bytes / `open(p,'wb')`), picking the newline from the file itself (`b"\r\n" if b"\r\n" in b`).
Check `git diff --stat` after every scripted edit: a 2-line change must report 2 lines.
An exact-match edit failing on such a file is usually a tab-vs-spaces mismatch, not a
missing line — dump the line with the whitespace made visible before assuming it moved.

เชื่อม [[reference_bak_restore_drops_comments]] [[reference_oc2plus_member_frontend]]
