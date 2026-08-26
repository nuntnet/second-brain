---
name: reference-lit-react-node-condition-hollows-tests
description: "@lit-labs/react resolves an SSR build under vitest that sets NO props on DS elements — every page test silently proves nothing until the browser export condition is pinned"
metadata:
  node_type: memory
  type: reference
---

`@lit-labs/react`'s package exports declare a **`node` condition** pointing at an SSR build whose `createComponent` emits the custom-element tag but **sets none of its properties**. Vitest runs in Node, so it resolves that build by default — and every `<ssk-*>` component renders bare: no `value`, no `helperText`, no `status`, no `testId`.

**Nothing throws.** Assertions read `undefined`, a page rendering its error on the wrong field looks identical to one rendering it correctly, and the suite goes green having proven nothing. Same failure class as a CI job that runs `lint && type-check` under the name `unit_test`.

**Fix** (ai-chat-admin-frontend `apps/admin/vitest.config.ts`): pin the browser condition in BOTH places —
```ts
resolve: { conditions: ["browser", "development", "import", "module", "default"] },
ssr: { resolve: { conditions: [...same] } }
```
Guarded by `apps/admin/src/test/dsBridge.test.tsx`, which renders one `SskInput` and asserts the props arrive. If that test ever fails, do NOT rewrite the page tests around it — the bridge is broken and all of them have stopped meaning anything.

**How to apply:** any repo bridging Lit web components into React tests (bola-frontend uses the same `createComponent` pattern) needs the same pin. Symptom to recognise: `getByTestId` finds nothing and the DOM dump shows `<ssk-button></ssk-button>` with zero attributes. See [[reference_ds_testid_is_a_property]] for the related query trap and [[reference_ssk_components_docs]].
