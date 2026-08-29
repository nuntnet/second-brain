---
name: rag-core-visibility-tiers
description: "rag-core has 3 visibility tiers (public/internal/private); internal reaches customers only via chat-core's employee impersonation and will break silently when fixed"
metadata:
  type: reference
---

Read from rag-core source 2026-08-29 (`feature/AI-46-retrieval-acl`).

`Visibility = Literal["public", "internal", "private"]`
(`src/entity/knowledge/chunk.py`), filtered in
`src/use_case/internal/knowledge_visibility.py`:

| tier | who reads it |
| --- | --- |
| `public` | everyone, including a conversation answering a customer |
| `internal` | **`employee` actors only** — `may_read_internal` = `actor_type in {"employee"}` (`workspace_scope.py:52`) |
| `private` | **the one customer who owns it** — `owner_customer_id == reader.reading_for`, nothing to do with employee |

So `private` IS the "external personal data" tier — no fourth tier is needed.
chat-core does pass `reading_for` (`rag_source/http.go:86,124`).

Two rules in `filter_by_visibility` worth preserving: **both sides must be
present AND equal** for private (a `None == None` comparison would hand a
private chunk to everyone), and an **unrecognised visibility is withheld**.

## 🔴 The trap

rag-core's create default is `internal`, and chat-core's retrieval presents as
an `employee` because **rag-core has no service actor type**. So `internal`
chunks reach customers TODAY by way of that impersonation. The day rag-core
grows a real service actor — the correct fix — every `internal` entry silently
stops reaching customers, and the symptom is an assistant that has "forgotten"
knowledge nobody deleted.

⇒ chat-core's KB BFF (AI-96) defaults to **`public`** and **refuses `private`**
(an admin typing in a console has no owner to attach; a private chunk with no
owner is readable by nobody at all).

Related: [[chatcore-is-the-admin-bff]], [[ai-conversation-intelligence]]
