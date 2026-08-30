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

**AI-149 (quota banner) — NOT blocked; I got this wrong too, the same way.**
I searched only chat-core, found `QuotaGate` unwired there (default
`allowAllGate`), and concluded no hard cap could fire. True about chat-core,
wrong as a conclusion: **the whole quota model lives in sellsuki-ai-agent**,
branch `feat/AI-82-hard-cap-gate` — `src/entity/quota_status/` has
`level` (green/yellow/red/exhausted/**rate_limited**), `used`, `ceiling`,
`used_percent`, `reset_at`, and `ai_disabled_since`. chat-core MR !46
(`feat/AI-155-quota-status-proxy`) proxies it at
`GET /v1/companies/:company_id/quota/status` with the tenant check the agent
cannot make. FE built 2026-08-30 (`d8058ae`); both backends still unmerged, so
it answers 503 and renders no indicator until they land.

Traps found building it: quota is metered per **company**, not per workspace
(one pool, several workspaces). `rate_limited` is NOT a hard cap — it clears in
seconds. `ceiling` is nullable and must stay null, never 0 (0 = no capacity,
the opposite of unmetered). And there is **no quota event on any
browser-facing transport**, so a live indicator is impossible today — poll.

**AI-148 (unmapped fact queue) — blocked on AI-7 + AI-16.** Needs an extraction
path and a schema to compare against; neither exists. `FactSchemaSource.ListFieldKeys`
is declared twice and **called** (`lead_rule/config.go:44`, `lead_data/gate.go:48`)
but has only mocks — no production adapter. No fact WRITE path exists anywhere
(`SetFacts`/`WriteFact`/`UpsertFact` all empty in chat-core and the kit).

**AI-183 — blocked on entity!43** (rps rejects `sellsuki.chat_workspace` tenant
kind). See [[ai150-members-read-only]], [[entity-lib-tenant-kinds]].

**AI-108 / AI-110 / AI-112 / AI-119** — PWA/Capacitor placeholder, billing read
model, provider dashboard. No backend anywhere; platform-level scope.

Method that mattered: I declared three "no backend" blockers and TWO were wrong
(AI-193 and AI-149). Both times the code was in a repo I had not searched.
**Search every submodule, not the one you are standing in** — and check
checkout branches: `sellsuki-ai-agent` was sitting on `feat/AI-82-hard-cap-gate`
the whole time.
Grep the PORT and its call sites, not just the repository directory — a port
with real callers and only a mock implementation looks identical to "missing"
until you check which. [[verify-absence-claims]],
[[head-on-grep-is-sampling-not-verification]].
