---
name: reference-workflow-doc-outline-and-plugin-regen
description: "The team-facing workflow guide lives in Outline (docs.sellsuki.com, id agentic-sellsuki-workflow-eXzL2RoN2H); how to refresh it and the plugin without false drift"
metadata:
  node_type: memory
  type: reference
  originSessionId: f13f419a-2bdd-4f04-b456-a56c1c7de007
  modified: 2026-11-03T15:50:12.120Z
---

The PO/Dev guide "Agentic Sellsuki Workflow" is the Outline doc `agentic-sellsuki-workflow-eXzL2RoN2H`
(https://docs.sellsuki.com/doc/agentic-sellsuki-workflow-eXzL2RoN2H). It lags `.claude/` and must be
refreshed by hand; rewritten 2026-11-03.

- `mcp__outline__update_document` text must NOT start with `# <title>` — Outline already renders the
  title, so an H1 shows the title twice.
- Plugin drift: run `python3 scripts/build-plugin.py --check [--json]` (then without `--check` to
  rebuild). Do not diff `.claude/commands/*.md` against `plugins/.../SKILL.md` by hand — frontmatter
  always differs, so every file looks "stale". 5 skills are hand-written and preserved
  (ac-tests, card, fanout, prove-red, sharpen).
- When a rule changes, both the Outline doc and the plugin need the follow-up; see [[project-po-workflow-pack-not-on-main-2026-11-02]].
