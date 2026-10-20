---
name: feedback_rtk_cat_strips_comments_never_use_cat_to_rewrite_files
description: "The rtk hook rewrites `cat` and strips code comments from its output — `cat x > f` silently deleted 13 comment lines from a Go file (2026-10-20). Rewrite files with python bytes, never cat"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 7fb71c5f-6e71-4687-b858-9caa035598f1
  modified: 2026-10-20T10:14:21.061Z
---

2026-10-20: to retrigger air I ran `cp f $TMPDIR/m.go && cat $TMPDIR/m.go > f`. The rtk hook rewrote `cat` to its filtered reader, which drops comments, so `main.go` lost 13 comment lines. Only `git status` showing `M` caught it. Restored with `git checkout --`.

**Why:** rtk's output filters are meant for reading, not for piping into a file. Their output is not the file's bytes.

**How to apply:** never use `cat`, `head` or `sed -n` output as file content in a redirect. To rewrite identical bytes (the air retrigger in [[reference_docker_desktop_half_dead_after_disk_full]]), use `python3 -c "p='f'; b=open(p,'rb').read(); open(p,'wb').write(b)"`, or `rtk proxy cat`. Afterwards check `git status --untracked-files=no`. Related: [[feedback_compare_command_outputs_with_rtk_proxy]].
