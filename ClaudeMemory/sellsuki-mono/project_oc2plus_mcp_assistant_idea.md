---
name: project-oc2plus-mcp-assistant-idea
description: 2026-09-27 user exploring an OC2Plus MCP / AI assistant for backoffice staff because the app is hard to use — idea stage, direction discussed
metadata:
  type: project
---
2026-09-27: user is exploring making OC2Plus easier via an MCP server + LLM (or a skill driving browser/computer use). Still at the idea stage. Nothing has been built or decided.

Direction discussed so far:
- Audience is backoffice staff, not members.
- Start with skill + browser, used internally by CS/onboarding. Promote only the frequently used tasks to MCP tools.
- MCP lives on the backoffice-api side, preferably as an in-process `/mcp` route. It does not live in sellsuki-ai-agent (decision-only, no side effects) or ai-platform-kit-go (a library; its `authctx` checks `chat_workspace`, not `sellsuki.company`).
- ACL: forward the user's identity, never a service key. Auth via OAuth/Hydra. Effective permission = OAuth scope ∩ Keto. Write tools use a server-enforced two-step preview/commit.

**Why:** the user may return to this in a later session. The reasoning above should not be re-derived from scratch.

**How to apply:** before building, verify two things: where the gateway sets `X-User-Id` (backoffice-api trusts that header blindly) and whether Hydra is deployed per env. Related: [[project-user-pain-evidence-gap]].
