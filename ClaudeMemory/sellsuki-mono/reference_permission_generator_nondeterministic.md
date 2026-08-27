---
name: reference-permission-generator-nondeterministic
description: entity's cmd/populate_permission emits vendor blocks in map order — regenerating produces a ~260-line diff that changes nothing
metadata:
  type: reference
---

`gitlab.sellsuki.com/sellsuki/sellsuki/backend/entity` generates
`access_control/permission_list.go` from `cmd/populate_permission`. The
generator iterates a Go map, so the four vendor blocks (Sellsuki / Patona /
Oc2Plus / Akita) come out in a **different order on every run**. Measured: two
consecutive runs gave two different orderings, and neither matched the committed
file on the first try.

Consequence: adding one permission can produce a ~260-line diff that changes
nothing semantically, which makes the MR unreviewable and hides the actual
change.

Also: the committed file is **not gofmt'd** (4-space indent, generator output).
Running `gofmt -w` on it reformats all ~500 lines. Don't.

Workflow that keeps the diff to the lines actually added:

1. edit `entity_oc2plus.go` (the entity constant) and
   `cmd/populate_permission/main.go` (the entity→actions entry)
2. `go -C cmd/populate_permission run main.go`
3. compare the block order against the committed file; re-run until it matches
4. verify with `git diff` — it should be only the new lines
5. never gofmt the generated file

Cross-check the semantic change independent of ordering:
```
git show HEAD:access_control/permission_list.go | grep -oE 'Permission[A-Za-z0-9]+ = "[^"]+"' | sort > /tmp/before
grep -oE 'Permission[A-Za-z0-9]+ = "[^"]+"' access_control/permission_list.go | sort > /tmp/after
diff /tmp/before /tmp/after
```

Worth fixing at the source by sorting the groups in the generator — until then
every contributor hits this.

The entity repo is **not a submodule** of sellsuki_mono; clone it separately.
Adding a scope is a 3-step release: merge → tag → bump the consumer's `go.mod`,
then ops must grant the scope to a role or the feature is dead on arrival.

Related: [[reference-entity-lib-tenant-kinds]], [[project-oc4362-claim-cluster-gaps]]
