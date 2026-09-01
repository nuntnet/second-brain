---
name: search-the-whole-stack-not-one-layer
description: "A negative finding needs every layer and every repo checked — I called four things blocked/broken after searching only one, and all four were wrong"
metadata:
  type: feedback
---

Distinct from [[verify-absence-claims]], which is about tools that lie when
you search. This is about **searching the right place and stopping too early**:
the grep was correct, the repo was correct, and the conclusion was still wrong
because the answer lived one layer up or one repo over.

Four instances in a single session (2026-08-30/31), all the same shape:

| Claim I made | Where I looked | Where the answer was |
|---|---|---|
| "AI-193 blocked, chat-core has no fact store" | chat-core `src/repository/` | `ProfileCardSource.GetFacts` + `IsFactVisible`, already called on every inbound message (`use_case/checkpoint.go:150`) |
| "AI-149 blocked, no hard cap possible" | chat-core (`QuotaGate` unwired → `allowAllGate`) | the entire quota model in **sellsuki-ai-agent** `src/entity/quota_status/` |
| "AI-82 not merged" | `git merge-base --is-ancestor <branch> main` | the branch had one extra docs commit; **the code was on main** |
| "1 workspace can have many FB pages" | messaging-backend DB schema (workspace_id index is non-unique) | the **use case** above it refuses `len(bindings) > 1` with a 409 |

**Why each fooled me:** every one had a true observation attached. The gate
really is unwired in chat-core. The branch really isn't an ancestor. The index
really isn't unique. The observation was right and the *scope* of the
conclusion was wrong.

**How to apply — before writing "blocked" / "missing" / "impossible":**

1. **Search every submodule, not the one you are standing in.** The AI platform
   is chat-core + ai-agent + ai-platform-kit-go + messaging-backend + entity;
   a capability can live in any of them.
   ```bash
   for r in backend/*/; do echo "== $r"; git -C "$r" grep -l "<symbol>" origin/main -- src 2>/dev/null; done
   ```
2. **Check the code on `main`, not the branch's ancestry.** `merge-base
   --is-ancestor` answers a question about commits, not about whether a feature
   shipped. `git ls-tree origin/main` / `git grep origin/main` answer the real one.
3. **A DB constraint is not the invariant.** The use case above it may enforce
   more (or less). Read both before describing what the system permits.
4. **Check dependency versions before believing a "waiting on upstream".**
   AI-183's blocker had expired a week earlier — see
   [[e8-remaining-blockers]].

Related: [[verify-absence-claims]],
[[head-on-grep-is-sampling-not-verification]], [[ground-claims-file-line]],
[[backend-without-frontend-is-invisible]].
