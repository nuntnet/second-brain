---
name: reference_staging_th_cluster_upstream_dns_timeouts
description: "staging-th CI jobs fail at get_sources with 'Could not resolve host: gitlab.sellsuki.com' — CoreDNS is healthy and idle but its UPSTREAM resolver 10.21.0.2:53 times out; 2 replicas, 774 pods, no NodeLocal DNSCache"
metadata:
  node_type: memory
  type: reference
---

Seen 2026-09-22 on `teleport.internal.staging-th.sellsuki.com`.

Symptom in CI — the job dies in `get_sources`, before a single line of the
script runs, so it looks like a script failure and `failure_reason` says
`script_failure`:

```
fatal: unable to access 'https://gitlab.sellsuki.com/…git/':
       Could not resolve host: gitlab.sellsuki.com
ERROR: Job failed: command terminated with exit code 1
```

**It is intermittent, not per-project.** On the same runner
(`Staging-th Runner General`, namespace `share`) jobs 252361/252362 passed at
12:23 and 252385/252391/252392 all failed after 13:12, while a different repo's
`_amd`/`_arm` jobs kept running. Retrying reproduces it exactly — so retry once,
then stop and treat it as infra (`shipping.md` §3).

What the cluster actually shows:

| check | result |
|---|---|
| CoreDNS pods | 2/2 Running, 23h, 0 restarts |
| CoreDNS CPU / mem | 30m, ~38Mi (limit 170Mi mem, **no CPU limit**) — idle |
| nodes / pods | 9 / 774 |
| NodeLocal DNSCache | **not deployed** |
| CoreDNS logs, 3h | **145** timeout/SERVFAIL lines |

The errors are all upstream, never internal:

```
[ERROR] plugin/errors: 2 …: dial udp 10.21.0.2:53: i/o timeout
[ERROR] plugin/errors: 2 …: write udp 10.21.37.145:54041->10.21.0.2:53: i/o timeout
[WARNING] plugin/health: Local health request to "http://:8080/health" failed: context deadline exceeded
```

`10.21.0.2` is the AWS VPC resolver. CoreDNS is not saturated — it is **blocked
waiting on upstream**, which is also why its own health probe times out while
the pod still reports 1/1 Ready. So "CoreDNS is Running" proves nothing here.

Most likely cause (**hypothesis, not measured**): the AWS VPC DNS hard limit of
1024 packets/sec per ENI. With no NodeLocal DNSCache, every upstream lookup from
774 pods funnels through the 2 nodes running CoreDNS, and `ndots:5` turns one
lookup into 4–5 queries. Over the cap, AWS drops packets silently — exactly this
signature. Confirming it needs the node's `linklocal_allowance_exceeded` ENA
metric, which I did not read. The fix that follows is NodeLocal DNSCache or
more CoreDNS replicas + longer cache TTL — an SRE change, not ours.

This is almost certainly also what hung `build_merge_request` #252146 in
`npm install` earlier the same day (npm makes many lookups), which then held the
`build` resource_group and froze every build in the project for eight hours —
see [[reference_stuck_ci_job_holds_resource_group]].
