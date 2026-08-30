---
name: e8-remaining-blockers
description: "AI-9/E8 leftover cards — which are genuinely blocked and on what, verified against code on 2026-08-30"
metadata:
  type: project
---

After the 2026-08-30 sweep (AI-96/99/103/106/129/150/175/182/193 all built
locally on `feat/AI-129-playbook-history` + `feat/AI-96-kb-entries`), these
remain. Each blocker verified in source, not inferred from card status.

**AI-193 — I first called this blocked and was WRONG.** `ProfileCardSource.GetFacts`
and `context_assembler.IsFactVisible` both existed and were already used on the
answer path (`use_case/checkpoint.go:150`). Only the browser route and the audit
were missing; both now built. What genuinely remains is AI-7's production
adapter (endpoint answers 503 without it) and `provenance`/`confidence`/
`schema version`, which are not fields on `context_assembler.Fact`.

**AI-149 (quota banner) — blocked on AI-82.** `ai-platform-kit-go/llmclient/gate.go`
declares `QuotaGate`, and chat-core wires **no adapter**, so the default
`allowAllGate` applies — the kit's own doc calls this "the honest state of the
world until AI-82 ships an adapter". No hard cap can fire, so the banner would
describe an impossible state. `degrade.ReasonQuotaExhausted` exists but comes
from a provider **402** per request (`llmclient/transport.go:74`) — not a
readable per-workspace state, and there is no "limit" value anywhere.
See [[quota-not-feature-gate]].

**AI-148 (unmapped fact queue) — blocked on AI-7 + AI-16.** Needs an extraction
path and a schema to compare against; neither exists. `FactSchemaSource.ListFieldKeys`
is declared twice and **called** (`lead_rule/config.go:44`, `lead_data/gate.go:48`)
but has only mocks — no production adapter. No fact WRITE path exists anywhere
(`SetFacts`/`WriteFact`/`UpsertFact` all empty in chat-core and the kit).

**AI-183 — blocked on entity!43** (rps rejects `sellsuki.chat_workspace` tenant
kind). See [[ai150-members-read-only]], [[entity-lib-tenant-kinds]].

**AI-108 / AI-110 / AI-112 / AI-119** — PWA/Capacitor placeholder, billing read
model, provider dashboard. No backend anywhere; platform-level scope.

Method that mattered: I declared two "no backend" blockers and one was wrong.
Grep the PORT and its call sites, not just the repository directory — a port
with real callers and only a mock implementation looks identical to "missing"
until you check which. [[verify-absence-claims]],
[[head-on-grep-is-sampling-not-verification]].
