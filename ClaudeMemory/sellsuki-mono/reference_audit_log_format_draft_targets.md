---
name: reference-audit-log-format-draft-targets
description: "New audit log format (Outline \"audit-log-draft\", 2026-09) — data.audit with result, request, targets[] {entity, entity_refs, entity_action, change{before,after}}; sellsuki-go-logger v2.1.1 does NOT support it yet"
metadata:
  node_type: memory
  type: reference
  originSessionId: 80f3d634-2df8-4993-a006-dce965f79cd9
  modified: 2026-09-30T03:09:57.502Z
---

Audit logs go to stdout (log_type=audit) → Loki. The format the user gave on 2026-09-30
(Outline `https://docs.sellsuki.com/doc/audit-log-draft-3PNoHlcJDJ`, needs login; Outline MCP often times out — ask user to paste):

```
{ level, timestamp, caller, message:"use_case.<Func>", app_name, version, alert, log_type:"audit",
  data: { audit: { actor_type, actor_id, result:"success"|..., action:"update",
                   request:{...input...},
                   targets:[ { entity:"oc2plus.crm.campaign", entity_refs:"CMP_…" (string, not array),
                               entity_action:"access|create|update|delete",
                               change:{ before:{…}|null, after:{…}|null } } ] },
          tracing: { span_id, trace_id } } }
```

One use-case call = one audit line with MANY targets (reads as `access`, side-effect rows as create/delete).

**Gap vs code (verified 2026-09-30):** `sellsuki-go-logger/v2` latest tag is v2.1.1 and its
`log.AuditPayload` is still flat (actor, action, entity, entity_refs[], entity_owner_type/id) —
no result/request/targets/change. PAT-2612 (S1 standard) still says "enforce v2 AuditPayload".
So the new format needs a logger release first.

**Draft has no entity_owner_type/id** → PAT-2614's Loki query tenant scoping (by owner) has nothing to filter on. Also `request`/`change` can carry PII — needs a redaction rule.

Supersedes the "closed enum + first EntityRefs marker" workaround in [[reference-audit-action-is-closed-enum]] once the logger supports targets. Epic: [[project_central_audit_log]].
