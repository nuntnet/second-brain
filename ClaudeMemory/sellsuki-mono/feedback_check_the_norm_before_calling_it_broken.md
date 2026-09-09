---
name: feedback-check-the-norm-before-calling-it-broken
description: "before reporting something as a defect, check whether its siblings look the same — a convention framed as a gap sends people fixing the wrong thing"
metadata:
  node_type: memory
  type: feedback
---

I reported "the migration repo has no CI, migrations are run by hand" as if it
were a broken repo. The user replied: **"ปกติงานของ oc2plus จะมี migration repo"**
— i.e. that is simply how OC2Plus works. Checking the siblings took one call and
showed all six OC2Plus migration repos have zero pipelines.

**Why:** framing a convention as a defect points the reader at the wrong
problem. The real gap was narrower and more useful — nothing connects "migration
MR merged" to "someone ran it on env X": no pipeline, no checklist in the service
MR, no deploy gate.

**How to apply:** before calling a missing piece a defect, compare against its
siblings (other repos of the same kind, other services in the same group). If
they all look that way, describe it as the convention and name the *specific*
consequence instead. Related: [[feedback-search-before-declaring-gap]],
[[feedback-verify-absence-claims]].
