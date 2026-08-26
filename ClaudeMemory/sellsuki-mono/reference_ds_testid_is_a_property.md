---
name: reference-ds-testid-is-a-property
description: "testId on an ssk-* component is a Lit property, never a data-testid attribute — getByTestId finds nothing and it looks exactly like the element not rendering"
metadata:
  node_type: memory
  type: reference
---

`testId` on a `@sellsuki-org/sellsuki-components` element is a **declared Lit reactive property**. `@lit-labs/react` assigns it to the DOM object, so **nothing appears in the markup**:

```html
<ssk-button>Connect</ssk-button>   <!-- testId="facebook-connect" is on the object -->
```

So `screen.getByTestId(...)` and `[data-testid="..."]` selectors find nothing — and that failure is indistinguishable from the element not being rendered at all.

**The inconsistency is the trap:** plain React components in ai-chat-admin-frontend (`ErrorState`, `PagePane`) DO forward `testId` to `data-testid`. The same query works for part of a page and silently fails for the rest.

**How to apply:** use `apps/admin/src/test/dsQueries.ts` (`dsByTestId` / `getDsByTestId` / `alertMessages`) for anything `ssk-*`, Testing Library's own queries for the rest. Read the element's PROPERTY (`el.helperText`, `el.status`, `el.disabled`, `el.message`) — asserting the page set the right property on the right DS component IS the page's contract with the design system.

Two related mistakes made while learning this (2026-08-25): counting *enabled* `ssk-button`s to test a permission gate is wrong on any page with modals — modal buttons render into the DOM whether open or not; assert the action row is ABSENT instead. And a page under AppShell reads `{session, workspaces}` from `useOutletContext`, so rendering it in isolation throws — `renderWithProviders` takes an `outletContext` rather than mocking the hook, because mocking it would stop the page's own permission derivation from being exercised.

Depends on [[reference_lit_react_node_condition_hollows_tests]] — without that pin every property reads `undefined` anyway.
