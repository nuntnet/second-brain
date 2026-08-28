---
name: silent-semantic-merge-break
description: "Two green MRs can break main on merge — a signature change and a new call site auto-merge cleanly and then fail to compile; dry-run the merge, don't reason about it"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 270e02aa-6ae1-461a-ada0-6fb19a8e9e87
  modified: 2026-08-27T08:09:16.127Z
---

Two MRs whose pipelines are both green can still break `main` the moment the
second one merges, with **no conflict reported by git**. Hit on 2026-08-27 in
`backend/sellsuki-chat-core` with !28 (AI-40 anti-abuse) and !29 (AI-192
delivery status):

- !29 added a `messageID` parameter to `sendReplyBestEffort`.
- !28 independently added a **new call site** for that function (the AI-40
  rate-limit cooldown reply), written against the old signature.

The edits sit in different regions of `src/use_case/chat_reply.go`, so git
auto-merges both without a murmur and emits `not enough arguments in call to
uc.sendReplyBestEffort`. Each branch compiles perfectly alone — **CI cannot
see this before the merge**, because no pipeline ever builds the combination.

The same pair also produced an ordinary reported conflict in
`cmd/migration/migrations/migrations.go` (both append registrations). That one
is the harmless kind: git tells you, and the resolution is "keep both, in id
order".

**How to apply — dry-run the merge, never reason about it:**

```bash
git worktree add --detach "$WT" origin/main
cd "$WT"
git merge --no-edit origin/<branch-A>
git merge --no-edit origin/<branch-B>
go build ./... && go vet ./... && go test -count=1 ./src/...
git worktree remove --force "$WT"
```

A reported conflict is the *easy* outcome. The signal to hunt for is a clean
merge that fails `go build` — so **always build and test the merged tree**,
not just check whether git complained. Reviewing the overlapping file list
(`comm -12` of each branch's `diff --name-only origin/main...`) tells you a
collision is *possible*, never that it is safe.

**Who fixes it:** neither branch can carry the fix while the other is still
unmerged — merging `main` into either one brings nothing that breaks. The fix
belongs to whichever merges **second**, which makes it exactly the kind of
thing that gets lost. Post the exact resolution as a comment on both MRs
before either lands.

Related: [[parallel-sessions-duplicate-symbols]] (clean auto-merge ≠ correct,
same root cause), [[ai-chatcore-merge-order]], [[ai-merge-topology-risk]].

## Text-level "keep both" is wrong for structured files

Same dry-run technique applied to `frontend/ai-chat-admin-frontend` (!10/!11/!12/!13,
2026-08-27). There the collision was purely additive — everyone appends to the
same registries — but two files still cannot be resolved by pasting the sides
together, because **git's hunk boundaries do not respect the file's structure**:

- **`i18n/locales/*.json`** — the hunk cuts through a nested object, so the two
  sides join at mismatched nesting and the file stops parsing (`Expecting ','
  delimiter`). Parse all three stages (`git show :1:/:2:/:3:`) and **deep-union
  as data**. Then assert: no key lost from either side, and **EN/TH at exact
  parity** — a silently dropped i18n key is an untranslated string in
  production, not a build failure, so nothing downstream will catch it.
- **`orval.config.ts`** (any config-object file) — hunks split mid-entry, so a
  textual keep-both yields syntactically plausible garbage. Rebuild from stages:
  take the top-level entries each side added over the base, splice into one.
  Assert the expected final count (there: 13 clients = 7 base + 2 + 4).

**Check what the type-checker cannot see.** After a registry merge, verify every
declared port/handler is still *wired*, not just still *declared* — losing the
wiring while keeping the interface type-checks clean and is simply dead at
runtime (`ports.tsx` declares vs `main.tsx` passes).

FE verification set: `pnpm type-check`, `pnpm lint`, `pnpm test`.

**GitLab does not auto-retarget stacked MRs in this group.** When a base branch
merges, the MR stacked on it keeps pointing at the dead branch and still reports
`mergeable` with a green pipeline. Re-point by hand; check every stacked MR.

**CSS belongs on that list too, and its verification set does not.** Resolving
`inbox.css` by concatenating both sides of each hunk produced two rules whose
closing brace was swallowed (`.bubble__delivery-badge--pending {` immediately
followed by the next selector) and one rule that lost its declarations
outright. **`pnpm type-check`, `pnpm lint` and all 274 unit tests passed on the
broken file** — none of them parse CSS. Only `pnpm build` failed, at postcss
`Unclosed block`. So: a frontend merge touching stylesheets is unverified until
`pnpm build` has run. Rebuild CSS as a union of whole RULES (main's file as the
base, append the rules only the branch has), then assert braces balance and no
selector from either side went missing. Duplicate selectors that already exist
on main are not yours to fix — check before "cleaning" them.

**An `add/add` test file is the same trap wearing a different hat.** Two lines
each created `MessageBubble.test.tsx`; git reported add/add, no content
conflict, and neither side could see the other. After merging both suites the
real incompatibility surfaced: main's tests render the component without a prop
this branch made REQUIRED. Nothing flagged it until tsc saw both files at once.
Merge both suites, assert the test count equals the sum, keep the other side's
tests verbatim, and adapt only what the type checker forces.
