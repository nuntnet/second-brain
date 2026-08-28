---
name: testify-permissive-default-wins
description: "chat-core test helpers pre-stub IsHumanMode/MarkDisclosureSent permissively; testify honours the FIRST matching .On() so a test's own stub is silently ignored and the test passes whatever the code does"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 270e02aa-6ae1-461a-ada0-6fb19a8e9e87
  modified: 2026-08-28T03:22:02.055Z
---

`backend/sellsuki-chat-core`'s shared test helper `newChatReplyUseCaseWithTier0Rules`
(reached via `newChatReplyUseCase`) registers **permissive defaults** at
construction time:

```go
sessionRepo.On("IsHumanMode", mock.Anything, mock.Anything, mock.Anything).Return(false, nil).Maybe()
sessionRepo.On("MarkDisclosureSent", mock.Anything, mock.Anything, mock.Anything).Return(false, nil).Maybe()
```

testify matches the **FIRST** registered expectation, so a `.On("IsHumanMode", …)`
written later inside a test **never fires**. The test then passes for the wrong
reason: not because the guard under test worked, but because it never saw the
value the test set. A guard that is entirely absent looks identical to one that
works.

How it surfaced on 2026-08-28 (AI-135, MR !32): a race-guard test stubbed
`IsHumanMode → (false, assert.AnError)` expecting fail-closed suppression, and
instead got `mock: Unexpected Method Call: AppendIdempotent(… "ext-img-4:attachment-only-reply")`
— the reply the guard should have suppressed. The panic points at the *reply
append*, not at the stub, so it reads as a bug in the guard.

**Two working fixes, both already in the repo — copy one, don't invent a third:**

- `sessionRepo.ExpectedCalls = nil` right after the helper, then re-register
  `GetOrCreateOpen` plus your own `IsHumanMode` (`chat_reply_outbound_test.go:224`).
  Simplest; use it for `chat_reply.go` guards.
- Build over caller-owned mocks via `newChatReplyUseCaseWithTier0RulesUsingRepos`
  (`reply_via_ai_agent_test.go:299`, whose doc comment explains exactly this).

`newChatReplyUseCase`'s own comment names the intended escape hatch but points
at `chat_reply_takeover_race_test.go`, **a file that does not exist** — don't
waste time looking for it.

**Prove red/green, and disable only the condition.** Deleting the whole guarded
block breaks the *build* (unused import), which proves nothing. Flip the
condition instead (`; suppress && false {`) and assert that **exactly** the new
tests fail and no others.

Related: [[timing-dependent-concurrency-tests]],
[[lit-react-node-condition-hollows-tests]] — same family: a green test that
never exercised the code.
