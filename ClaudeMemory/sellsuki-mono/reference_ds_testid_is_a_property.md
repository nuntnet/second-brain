---
name: reference-ds-testid-is-a-property
description: "testId on an ssk-* component is a Lit property — invisible in jsdom, and in a real browser it lands INSIDE the shadow root where its text is empty and clicking it does not reach the control"
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

## In a real browser (Playwright), the same property bites differently (2026-08-29)

Under a real DOM the DS *does* stamp `data-testid` — but on an element **inside its shadow root**, not on the host. Two consequences, each of which looked like a broken app rather than a broken selector:

- **Text assertions read whitespace.** The label is a slotted LIGHT-dom child of the host, so the shadow element carrying the testId contains only the `<slot>`. `toHaveText('[data-testid="lead-stage-x"]', "ติดต่อแล้ว")` receives `"\n \n"`. Assert on the plain `<li>`/`<div>` row that owns the component instead.
- **Clicking it does nothing.** A click on `[data-testid="notify-enabled-toggle"]` left `ssk-toggle.checked` false — the control never changed, so everything downstream was dead and the failure surfaced three assertions later as "Test send is disabled". `ssk-dropdown` is worse: it forwards no testId at all, exposes no button/input/combobox, and its slotted `ssk-dropdown-option`s are never visible.

**How to apply:** drive DS form controls through the element contract — set the property on the HOST and dispatch the mapped event (`SskToggle`/`SskDropdown` map `onChange` → `change`, and the app reads `e.target.checked` / `e.target.value`). That still exercises the app's own handler; whether the DS's shadow widget responds to a click is the DS's test, not the app's. Plain `SskButton` clicks are fine — those are ordinary DOM clicks on the host.

See `apps/admin/e2e/leads.spec.ts` (notify settings) for the worked example.
