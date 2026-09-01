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

**AI-148 — HALF the blocker is gone.** AI-16 shipped: the kit's `customerfact`
package has Schema, `SchemaService.ApplyPatch` (admin adds a field),
`WriteGate.Validate` returning **`UnknownFieldError`** (= the unmapped-fact
signal), HTTP routes, consent, encryption, audit. What is still missing is a
PRODUCER (AI-7 — nothing extracts facts and writes them through the gate) and a
store for the queue itself. Nothing imports `customerfact` yet.

⚠️ `chat-core/src/entity/context_assembler.IsFactVisible` DUPLICATES the kit's
`customerfact.IsFactVisible`, and a comment says to delete the local copy once
AI-16 merged. **Do not** — the local one takes `factCaseID`/`readerCaseID` and
the kit's does not, so deleting it drops Case isolation (AI-193 depends on it).
The kit needs Case scope first.

**OLD note:** Needs an extraction
path and a schema to compare against; neither exists. `FactSchemaSource.ListFieldKeys`
is declared twice and **called** (`lead_rule/config.go:44`, `lead_data/gate.go:48`)
but has only mocks — no production adapter. No fact WRITE path exists anywhere
(`SetFacts`/`WriteFact`/`UpsertFact` all empty in chat-core and the kit).

**AI-183 — DONE 2026-08-31, and the blocker had expired a week earlier.**
entity!43 merged and shipped in entity **v0.31.0** on 2026-08-24; **rps already
pinned v0.31.0**. Only the kit (still entity v0.22.0, `DefaultTenantKinds()`
Workspace empty) and chat-core were behind. Fixed: kit !13 merged and tagged
**v0.3.0**; chat-core !49 bumps entity+kit, no code change needed. authctx now
honours a workspace-scoped Operator grant; AI-150 rows flip to editable with no
UI change. ⚠️ pin ≠ deploy — rps must be RUNNING >= v0.31.0.

**OLD note (kept as the lesson):** was recorded as blocked on entity!43 (rps rejects `sellsuki.chat_workspace` tenant
kind). See [[ai150-members-read-only]], [[entity-lib-tenant-kinds]].

**AI-113 — IN PROGRESS 2026-09-01** (chat-core MR !50): entity + store + use
case + audit + route, mutation-checked. Key is `(workspace_id, year_month)`,
which is correct ONLY because messaging-backend refuses a workspace with more
than one FB Page binding (409, "the pilot's single-connection model") — if that
rule is ever lifted the key must grow an `fb_page_id` or two Pages' spend sums
into one figure. FE was already fully built against a mock; swapping it to the
real adapter is the last step and should wait for !50 to land.

**AI-119 — bundle/perf shipped** (frontend MR !28, −7.1%); push and
standalone-auth deferred by the user. See [[ai119-push-deferred]] and
[[ds-bundle-dominates-and-cannot-treeshake]].

**AI-108 — closed Done** by the user; its own Out of Scope said "no
implementation in the pilot".

**AI-110 / AI-112 — genuinely not ready, and not for technical reasons.**
AI-110 is `blocked-by` two API cards **that have never been opened** (OMS
billing history, SukiPay payment method) plus O3. AI-112 is a `platform-track`
placeholder with an unresolved decision in the card itself (provider view in
the AI admin panel vs in provider-management-frontend) and depends on the E9
warehouse (AI-97/98), which does not exist. Deciding AI-112's UI location now
would produce an answer that is forgotten before it can be used.

Method that mattered: I declared three "no backend" blockers and TWO were wrong
(AI-193 and AI-149). Both times the code was in a repo I had not searched.
**Search every submodule, not the one you are standing in** — and check
checkout branches: `sellsuki-ai-agent` was sitting on `feat/AI-82-hard-cap-gate`
the whole time.
Grep the PORT and its call sites, not just the repository directory — a port
with real callers and only a mock implementation looks identical to "missing"
until you check which. [[verify-absence-claims]],
[[head-on-grep-is-sampling-not-verification]].
