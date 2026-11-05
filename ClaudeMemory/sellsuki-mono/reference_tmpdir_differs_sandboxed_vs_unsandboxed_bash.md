---
name: reference_tmpdir_differs_sandboxed_vs_unsandboxed_bash
description: "$TMPDIR is /tmp/claude-501 in sandboxed Bash but /var/folders/.../T in unsandboxed Bash — files a nohup ladder writes under $TMPDIR unsandboxed look 'missing' from a sandboxed check; use the absolute path"
metadata:
  node_type: memory
  type: reference
  originSessionId: 98a4f6f3-8b50-4cf4-a16b-f79af63aa589
  modified: 2026-11-05T10:59:15.207Z
---

Seen 2026-11-05 (wave 3, AI-382/393): a chat-core ladder started with `dangerouslyDisableSandbox` wrote its logs and done-file to
`$TMPDIR/w3db/` = `/var/folders/hz/0472_d597nv11489_lnnj_8m0000gn/T/w3db`. A later sandboxed `ls $TMPDIR/w3db` resolved to
`/tmp/claude-501/w3db` and said "No such file or directory", which reads like the ladder died.

**How to apply:** when a background job runs unsandboxed (go test, glab, anything needing TLS or the go cache), write its
log/done-file to an absolute path, and check it with that same absolute path (or also unsandboxed). Before concluding a
ladder died, `pgrep -fl` it. Related: [[reference_background_ladder_dies_with_the_turn_use_nohup]],
[[reference_sandbox_blocks_go_build_cache_and_bsd_sed_noop_mutation]].
