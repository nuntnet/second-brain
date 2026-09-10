---
name: reference-review-bot-finding-right-reason-wrong
description: "The GitLab code-reviewer bot's FINDINGS are usually real, but its stated reason and suggested patch are often wrong — verify the premise and check the fix against the rest of the suite before applying it verbatim"
metadata:
  node_type: memory
  type: reference
---

Repeated pattern across BOLA/CCS MR rounds (2026-08). The bot is worth taking
seriously — it caught real bugs every round — but **applying its patch verbatim
introduced new bugs more than once**. Treat the finding as a lead and re-derive the
fix.

Concrete cases:

| Bot said | Reality |
|---|---|
| "`t.Parallel()` — the project standard requires it" | 5 occurrences in exactly one file, **zero** under `src/`, no rule file. Applied anyway, but corrected the record. |
| `!160`: the dead guard exists because "`NewKratosWithAdmin` is the only constructor used for any admin-capable instance" | There are **three** constructors; `NewKratos` does NOT normalise `AdminURL` and returns the same struct. The guard was only safe to delete after making that impossible. |
| `!108`: replace `Math.ceil` with `Math.floor` | Fixes the reported bug and **breaks an existing test** in the same MR ("in 3 days" → "in 2 days"). Elapsed-ms was the wrong unit entirely; the fix was a difference of midnights. |
| `!110`: "Option A is preferred, no handler change needed" | Hiding the tab is insufficient — `inviteMode` is React **state** and survives the mode resolving mid-session. Both A and B were needed. |
| `!161`: "your earlier fix commit did not address these" | **Correct** — that commit answered a different sub-review on the same ticket. Worth conceding plainly. |

Also: the bot marks things INFORMATIONAL that are worth doing anyway (a banner
heading contradicting its own body copy; a `!= nil` guard that made a cross-tenant
check optional *in behaviour*, not just in appearance).

**How to apply:** for every finding, (1) verify the stated premise with a grep/read
before believing it, (2) run the suggested patch against the whole suite not just
the new test, (3) reply saying which part you took and which part you rejected and
why. See [[reference_gitlab_review_bot_targets]] for when the bot runs at all.

**2026-09-10, backoffice-api !551 (second instance):** bot asked for `t.Parallel()` because it is "used universally in the surrounding test suite" — actually 18 of 80 `_test.go` files, and the nearest neighbours (`point_claim_test.go`, `campaign_test.go`, `campaign_product_scope_v1_test.go`) do not use it. Fix still worth making (those files touch no global; `-race` clean) → adopt the fix, correct the premise in the reply. Never let "the bot says it's the convention" stand in for `grep -rl` over the suite.
