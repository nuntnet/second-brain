---
name: reference_console_paging_footer_needs_canvas_scroll
description: ai-chat console ListPagingFooter sentinel fires only when scrolled into view inside main.main-landmark; rootMargin prefetch does nothing; e2e must scroll the canvas
metadata:
  node_type: memory
  type: reference
  originSessionId: bb596177-5d75-43de-9fa1-faac69b7628b
  modified: 2026-11-03T10:29:17.200Z
---

In `frontend/ai-chat-admin-frontend`, `ListPagingFooter` (keyset "load more" via IntersectionObserver, root = viewport, `rootMargin` 200px) sits inside `main.main-landmark`, which is its own `overflow: auto` scroller. `rootMargin` only grows the viewport, not the inner scroller's clip, so:

- the next page loads only when the sentinel is actually visible inside the canvas (no early prefetch), and
- Playwright `scrollIntoViewIfNeeded()` on the last row is NOT enough. Scroll the canvas: `page.locator("main.main-landmark").evaluate(m => m.scrollTo(0, m.scrollHeight))`.
- Unit tests (jsdom) never see this: there is no IntersectionObserver there, so the footer falls back to its button.

Also: the mock adapters sort like the server (`updated_at DESC, id DESC`). Seeds that share one timestamp fall to the id tie-break and reorder (lead_demo_5 first), which moves fixtures between pages and shifts a11y-baseline selectors. Give seeds distinct times.

Found 2026-11-03 on the AI-313 lead-board paging follow-up. Related: [[reference_console_e2e_port_5196_shared_use_temp_config]], [[reference_admin_fe_worktree_install_and_i18n_key_count_tripwire]].
