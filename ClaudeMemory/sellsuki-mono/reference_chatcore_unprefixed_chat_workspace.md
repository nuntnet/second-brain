---
name: reference-chatcore-unprefixed-chat-workspace
description: "chat-core's role_assignment_repository sends tenant kind 'chat_workspace' with no sellsuki. prefix — AssignOperator stays rejected even after rps deploys entity v0.31.0"
metadata:
  node_type: memory
  type: reference
---

`backend/sellsuki-chat-core/src/repository/role_assignment_repository/grpc.go` declares its own:

```go
const TenantKindChatWorkspace = "chat_workspace"   // NO sellsuki. prefix
```

but `backend/entity` v0.31.0 registered **`sellsuki.chat_workspace`** (AI-182). The unprefixed form is not in `IsActor`'s allowlist and never was.

**Consequence:** even after rps merges its entity bump (!107) and deploys, chat-core's `AssignOperator`/`RevokeOperator` will still fail `InvalidArgument` at `access_control.NewNamespace -> ValidateKind -> entity.IsActor`. This is the last thing between rps !107 and AI-183 actually working, and no card owned it as of 2026-08-25.

**Why the local const exists:** chat-core pins `ai-platform-kit-go v0.0.0-20260802163850`, which predates `authctx.TenantKindChatWorkspace` entirely, so it had nothing shared to import.

**How to apply:** the fix is small — bump the kit pin and use the shared const. The READ path added for AI-137 (`src/use_case/accessible_workspace.go`) already spells the **prefixed** value and carries a comment pointing at this mismatch, so do not "make them consistent" by copying the unprefixed one. Verify against [[reference_entity_lib_tenant_kinds]].
