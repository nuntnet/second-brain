---
name: head-on-grep-is-sampling-not-verification
description: "Piping grep through head -N on a long file samples it; concluding 'X does not exist' from a truncated result produced a wrong public report twice — count first, or read the whole match list"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 270e02aa-6ae1-461a-ada0-6fb19a8e9e87
  modified: 2026-08-28T05:03:40.877Z
---

`grep ... | head -20` on a long file is **sampling**, not verification. A
negative conclusion drawn from it ("no test covers X", "this repo has no Y") is
unsupported, because the evidence that would refute it is exactly what `head`
discarded.

2026-08-28, auditing AI-124's acceptance criteria in
`sellsuki-messaging-backend`: I ran `grep -nE "func Test|company" ... | head -20`
on a **485-line** test file and reported to the user and on two public MR notes
that AC6 (2 companies × 2 workspaces) and AC7 (no token plaintext) had no test
coverage. Both were fully covered. The AC6 test sat at **line 389** — past the
cut. What I actually saw was the top of the file, where every fixture happens to
use `company_A`.

The cost: a wrong status report to the user, two wrong public MR comments, and
I was one step from writing duplicate tests for coverage that already existed.

**How to apply — before asserting an absence:**

```bash
grep -c "pattern" file          # count first; if the count exceeds your head -N, do not head
grep -n "^func Test" file       # for "does a test exist", list ALL of them — this output is small
```

Prefer a listing that is *complete by construction* (all function
declarations, all struct fields) over a filtered one you then truncate. When a
match list is genuinely too long to read, that is itself the finding — say
"too many to enumerate", never "none".

**Second-order lesson: an absence claim deserves a second look before it goes
public.** Absence is the claim most likely to be a tooling artifact, and the
one that costs the most when wrong — it sends someone off to build what already
exists. Positive findings tend to be self-verifying (the thing is right there);
negative ones never are.

Related: [[feedback_verify_absence_claims]] (same failure via a lying clone
refspec), [[feedback_ground_claims_file_line]], [[reference_rtk_git_output_filtering]].

**Third variant, 2026-08-29 (`ai-platform-kit-go`): a command that ERRORED, read
as "no matches".** Checking whether AI-71 AC8 had an enforcing test, I ran

```bash
grep -rlniE "go list|deps.*redis" llmclient/ *_test.go
```

from a directory with no top-level `*_test.go`, so zsh failed the glob:
`(eval):3: no matches found: *_test.go`. **grep never ran.** The output pane was
empty, I read empty as "nothing found", and reported "no guard test exists" to
the user as a verified finding. Re-running it properly the next turn happened to
confirm the conclusion — which is luck, not method.

Empty output has at least three causes and they are not distinguishable by
looking: no matches, a shell error before the command ran, and rtk filtering
eating the result (seen again in the same turn — `grep -c` said 6 while the
listing of the same pattern printed nothing). Check `echo $?`, or redirect to a
file and `wc -l` it, before treating empty as evidence.
