---
name: reference_chat_core_pins_unmerged_kit_by_pseudo_version
description: "chat-core can build against an UNMERGED ai-platform-kit-go change: push the kit branch, `go get …/ai-platform-kit-go@<sha>` (GOPRIVATE already covers gitlab.sellsuki.com) → pseudo-version; swap for the tag before merging"
metadata:
  node_type: memory
  type: reference
  originSessionId: 3c6e4362-8fae-41c1-a18f-6e53fb5cee11
  modified: 2026-11-02T04:31:31.519Z
---

Used on AI-382 (2026-11-02). AI-316 had instead left chat-core pinned to the old kit, so its branch did not build in CI until the kit was tagged.

**How to apply:** after pushing the kit branch, in chat-core run `GOFLAGS=-mod=mod go get gitlab.sellsuki.com/sellsuki/share/library/ai-platform-kit-go@<kit sha>` (unsandboxed). go.mod gets e.g. `v0.8.1-0.20261102003426-5007a5e15094` and only that line changes. The chat-core MR must say to replace it with the released tag (one commit) after the kit MR merges and is tagged. Kit MRs are not squashed, so the commit stays reachable. Related: [[feedback_open_mr_only_after_full_test_ladder_green]].
