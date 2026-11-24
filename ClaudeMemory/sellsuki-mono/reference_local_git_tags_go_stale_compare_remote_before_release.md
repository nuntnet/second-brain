---
name: reference-local-git-tags-go-stale-compare-remote-before-release
description: "A submodule's local tag (e.g. CCS v1.8.0) can point at a different commit than the GitLab tag — `git fetch --tags` never moves an existing local tag. Diff against `git ls-remote origin refs/tags/<t>^{}` before deciding what a release ships"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-24T10:49:34.137Z
---

2026-11-24, before tagging CCS v1.8.1: `git log v1.8.0..origin/main` showed 263 merges / 1,462 commits (other teams' stock-adjust, slip2go, OC work), which read as "prod runs a hotfix line, tagging main ships everything". Wrong: the local `v1.8.0` pointed at a January commit (`08f1d635`); the real tag on GitLab was `f16e4064`, main minus only our merge. `git fetch --tags` does not overwrite a local tag that already exists, so the stale one survived.

**How to apply:** before any release diff, resolve the tag remotely: `sha=$(git -C <repo> ls-remote origin "refs/tags/<tag>^{}" | cut -f1)` (fall back to `refs/tags/<tag>` for a lightweight tag), then `git log --first-parent $sha..origin/main`. Never trust a local tag for "what is on prod". Also confirm against the running image (`kubectl ... -o jsonpath='{..image}'`). Related: [[project-production-ccs-runs-hotfix-line-not-main]] — that note is now outdated for CCS (prod v1.8.0/v1.8.1 are cut from main).
