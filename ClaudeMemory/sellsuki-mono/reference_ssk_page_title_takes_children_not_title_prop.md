---
name: reference_ssk_page_title_takes_children_not_title_prop
description: "Console `SskPageTitle` (lib/ds.tsx) renders its CHILDREN; `<SskPageTitle title=\"…\"/>` compiles and renders an empty h1 — unit tests stay green, the a11y/e2e `h1:visible` wait fails"
metadata:
  node_type: memory
  type: reference
  originSessionId: 9bfbf442-2ea7-482e-8e36-8246c00e89fd
  modified: 2026-11-02T21:02:44.965Z
---

Seen 2026-11-03 on AI-371 (ai-chat-admin-frontend, kit branch): four new pages used `<SskPageTitle title={t(...)} />`. tsc accepted it (ssk-heading has a `title` attribute), jsdom tests passed, every page had an empty h1. Caught only when checking the `.workspace-page__header` contract that `workspaceHeader.spec.ts` / `a11y.spec.ts` rely on.

**How to apply:** write `<div className="workspace-page__header"><SskPageTitle>{t("…")}</SskPageTitle></div>` like the AI-370 pipeline pages, and assert the heading text in a unit test (`document.querySelector(".workspace-page__header ssk-heading")?.textContent`).
Related: [[reference_merge_conflict_keep_both_drops_shared_closing_lines]].
