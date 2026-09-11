---
name: feedback_grep_the_spec_is_not_grep_the_service
description: "Absence check for an HTTP endpoint must grep the whole interface layer, not the OpenAPI spec — OC2Plus member-api registers BFF routes outside spec/v1.yaml, so a spec-only grep reported 4 endpoints missing that two open MRs already shipped"
metadata:
  node_type: memory
  type: feedback
---

**Burned 2026-09-11.** I concluded `GET /v1/me/point`, `/me/point/transaction`,
`/me/point/expire` and `/me/campaigns` existed "on no branch" of
`oc2plus-line-crm-service-member-api`, opened OC-4538 on that premise, and had an
agent build a second implementation — while **MR !105 (`feat/oc-4347-customer-bff`)
and !106 (`feat/bff-wave2`) already shipped the same routes** as a session-scoped
proxy to 3rdparty-api.

**Two compounding mistakes, both mine:**
1. I grepped only `src/interface/fiber_server/spec/v1.yaml`. That repo is
   OpenAPI-first for most routes but the BFF work registers handlers directly in
   `src/interface/fiber_server/route/me_customer_bff_v1.go`, outside the spec. A
   spec-only grep cannot see them.
2. The branch loop had `head -40` on a repo with **41** remote branches.

**Why:** "not in the contract file" is not "not in the service". An absence claim
about behaviour has to be checked against the code that implements the behaviour.
This is the same class as [[feedback_verify_absence_claims]] and
[[feedback_head_on_grep_is_sampling_not_verification]] — and the cost was higher
than usual because a wrong absence claim becomes a Jira card, and a Jira card
becomes duplicate engineering.

**How to apply:** before declaring an endpoint missing, run all three and reconcile
— `git grep -n "<path>" <branch> -- 'src/interface/**'` (never just the spec),
the branch loop **with no `head`**, and a check for open MRs
(`glab mr list -R <repo>`) whose titles name the feature. If the finding will
become a card, say which of those you ran in the card itself so the next person
can see the blast radius of it being wrong.

Related: [[reference_oc2plus_member_react_migration]]
