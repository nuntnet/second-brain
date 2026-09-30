---
name: reference_glab_api_host_follows_cwd
description: "glab api picks its host from the cwd's git remote — run from the scratchpad (or any non-GitLab dir, incl. a background script) and every call goes to gitlab.com and returns empty; set GITLAB_HOST=gitlab.sellsuki.com"
metadata:
  type: reference
---

`glab api "projects/<enc>/..."` does **not** know about gitlab.sellsuki.com by
itself: the host comes from the current directory's git remote. From the
scratchpad, `~/.claude/...`, the SecondBrain folder, or a background script
started there, the request goes to gitlab.com, the body is empty, and
`json.loads` fails with "Expecting value: line 1 column 1".

Cost on 2026-09-30: a background pipeline watcher (OC-4629 MRs) read nothing
for 50 minutes and timed out; it looked like "no news", not "broken".

**How to apply:** in any script or loop, `export GITLAB_HOST=gitlab.sellsuki.com`
(and pass it in `subprocess` env), or run from inside a submodule checkout.
Before leaving a watcher in the background, make one foreground call with the
exact same setup and confirm it returns jobs.

Related: [[oc2plus-ci-outage-2026-09]] (retry infra failures by job id, ~3×).
