---
name: reference-control-tower-data-js-diverged-main-vs-ai49
description: "docs/control-tower on root `main` lacks validate.js, the phase-4 epics section and the generated md snapshot that the AI-49 branch has; which version is canonical is undecided"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9043b5b7-a828-4cca-b908-2d5347a98fbf
  modified: 2026-10-07T01:41:59.733Z
---

Seen 2026-10-07 while cherry-picking SukiSpace docs commits from `fix/AI-49-safe-worker-activation` onto `origin/main`:
`docs/control-tower/data.js` conflicted heavily. On `origin/main` the directory holds only `config.example.json data.js
index.html server.py`; there is **no `validate.js`**, no phase-4 `epics:` section (the `init: "sukispace-patona-sale-channel"`
row does not exist), and `docs/sellsuki-roadmap.generated.md` is not tracked. The AI-49 branch has all of those plus
`data.schema.md`, `generate-roadmap-md.js`, `ct-*.sh`. The two control-towers are diverging and nobody has decided
which is canonical.

**How to apply:** before editing `data.js`, check which branch you are on and whether `validate.js` exists; an edit
validated on AI-49 may not even have an anchor on `main`. Resolve conflicts by taking `main`'s file and re-applying
edits, then at least syntax-check with `node -e` + `vm.runInContext`. Raise the canonical-version question in the MR
rather than silently picking one (done in monorepo MR !29). Related: [[project-control-tower]].
