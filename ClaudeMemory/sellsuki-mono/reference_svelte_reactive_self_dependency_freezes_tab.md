---
name: reference_svelte_reactive_self_dependency_freezes_tab
description: A Svelte 4 `$:` block that reads a variable it also assigns re-triggers itself; with an already-resolved promise it starves the event loop and hard-freezes the tab
metadata:
  type: reference
---

In Svelte 4 a `$:` block depends on every variable it *reads*. Reading one it
also assigns makes it its own trigger. The classic shape is an accumulate-into-a-map
pattern:

```js
translatedGroupLabels = { ...translatedGroupLabels, [code]: label };  // read + write
```

inside a `$:` block that also resets the map. Each resolved promise re-runs the
block, which clears the map and re-issues the promises.

**The part that is easy to get wrong: whether this spins or freezes depends on
the data.** If the promise is genuinely async (a network fetch) it merely loops
and the page still paints, so it reads as "slow" or "flickering". If the promise
is *already resolved* — a lookup helper that returns a fallback synchronously for
unknown keys — the whole cycle stays inside the microtask queue and the event
loop never gets a turn. No paint, no input, and devtools cannot even evaluate a
constant: every script injection and page read times out. The tab is gone, and
navigation away from it fails too, so you need a fresh tab.

Found in `frontend/sellsuki-invitation/src/pages/invitation/section/PermissionList.svelte`
(2026-09-11). `translatePermissionGroup` returns the code synchronously for any
permission group absent from `PERMISSION_GROUP_I18N_KEYS`, which is every OC2Plus
and Patona group — only the six Sellsuki ones are mapped. So the freeze looked
role-dependent, and the user's report was literally "ถ้ามี role ของ oc2plus จะพัง".
That data-dependence is the diagnostic: a hang that only some accounts see, with
a dead devtools, points here.

**Fix**: accumulate into a local `const` map and assign the reactive variable
exactly once, so the block reads nothing it writes. Keep a batch counter if you
relied on the spread for ordering — `const batch = ++labelBatch` and only assign
when `batch === labelBatch`, so a slow earlier language cannot land on top of a
newer one.

**How to prove it rather than assume it**: put the old block back and watch a
one-line synchronous script time out, then restore the fix in a fresh tab.
`npm run check` passes on both versions — a type check says nothing here. See
[[feedback_verify_as_the_user_sees_it]] and [[feedback_reproduce_the_number_before_asking]].
