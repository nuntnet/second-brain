---
name: reference-keto-staging-permission-lookup
description: "how to check what permissions a company's roles actually hold on staging — port-forward keto-read in ns share and read relation-tuples; turns a 403 guess into a fact"
metadata:
  node_type: memory
  type: reference
---

Keto lives in namespace **`share`** on the staging-th cluster (`keto-read` /
`keto-write`, ClusterIP :80 → 4466/4467). The services rps points at are
`keto-read.share:80` and `keto-write.share:80`.

To turn "probably a permission problem" into a fact, read-only:

```bash
kubectl -n share port-forward svc/keto-read 24466:80 &
# every permission held on one company, grouped by role
curl -s "http://127.0.0.1:24466/relation-tuples?namespace=permissions&relation=sellsuki.company:<COMPANY_ID>&page_size=200"
# who holds one specific permission
curl -s "http://127.0.0.1:24466/relation-tuples?namespace=permissions&object=<PERMISSION>&page_size=500"
```

Tuple shape: `namespace=permissions`, `object=<permission code>`,
`relation=<tenantRef>`, subject-set `roles / sellsuki.role:<id> / <tenantRef>`.

**How to apply:** quote the actual tuple count rather than inferring from a 403.
Used 2026-09-01 to show company `f2bf928c…` held 11 of 12 `oc2plus.*`
permissions and zero tuples for `oc2plus.apikey.manage` — see
[[project-ccs-role-presets-apply-only-at-creation]]. In zsh the URL **must** be
quoted (`?`/`&` glob), and rtk rewrites `curl`, so use `/usr/bin/curl` to see
real JSON instead of a schema summary
([[reference-rtk-git-output-filtering]]).
