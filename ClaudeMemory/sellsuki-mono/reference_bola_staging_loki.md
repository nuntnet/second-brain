---
name: reference_bola_staging_loki
description: How to query staging BOLA logs via Loki (label names + gateway) when a pod restarted and kubectl logs is gone
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
---

**Debugging staging BOLA when the pod already restarted (kubectl logs --previous empty):** logs are shipped to **Loki** in the `system-logging` namespace on the staging-th teleport cluster (`teleport.internal.staging-th.sellsuki.com-...` context).

Query it:
```
kubectl --context <staging-th> port-forward -n system-logging svc/loki-loki-distributed-gateway 3199:80 &
curl -sS -G http://localhost:3199/loki/api/v1/query_range \
  --data-urlencode 'query={container_name="backoffice-service"} |= "<search>"' \
  --data-urlencode 'start=<RFC3339>' --data-urlencode 'end=<RFC3339>' --data-urlencode 'limit=200' --data-urlencode 'direction=forward'
```

**Gotchas that cost time:**
- Loki label is `namespace_name` NOT `namespace`; there is NO value "bola" under namespace_name — use **`container_name="backoffice-service"`** (the bola-backend container is named `backoffice-service`, not `back-office-of-line-api-backend`).
- Other labels: `pod_name`, `pod_ip`, `host`, `cluster`, `job`. No `app`/`container` labels.
- Time: BOLA UI timestamps are Thai (UTC+7); Loki wants UTC (subtract 7h).
- Live tail alternative: `kubectl logs -n bola <pod> -f` (run_in_background) — but connection drops after a while; Loki is the durable source.

Live pod logs are JSON lines (`request_log` middleware) that include the full request incl. body — useful for seeing the exact webhook payload that failed. Related: [[project_bola_deploy_values_in_repo]].
