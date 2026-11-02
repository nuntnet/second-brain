---
name: feedback_read_review_bot_before_merging
description: "Before merging any MR, read the SRE review bot's verdict line; \"mergeable + green\" does not mean the bot passed it"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 8b69c115-30aa-4ff4-b9c5-86fafae17c77
  modified: 2026-11-02T06:29:28.175Z
---

On 2026-11-02 I merged kit !36 (AI-393) right after nunt said "Yes, merge kit". GitLab said mergeable and the pipeline was green. The bot's last line said "Ready to merge once Finding 1 is addressed", a SHOULD FIX for deleted doc comments that was true. It had to be fixed forward in a new MR.

**Why:** `/land` lists the bot harvest (step 3) after the merge (step 2), and `scripts/land.py --merge` checks draft, conflict, pipeline and merged state, but not the bot. So a merge can pass every check while the bot says not yet.

**How to apply:** before every merge, read the bot note's `### Result:` line and its Findings table. If it has a SHOULD FIX or MUST FIX, report it and fix it first; do not merge. Only INFORMATIONAL findings may be left for the harvest after the merge. Related: [[feedback_standing_approval_merge_deploy_staging_after_gates]].
