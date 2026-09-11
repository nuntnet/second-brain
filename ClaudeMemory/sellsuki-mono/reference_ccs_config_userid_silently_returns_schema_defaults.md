---
name: reference_ccs_config_userid_silently_returns_schema_defaults
description: central-configuration-system does not fall back to the global config when a userId has no override — it answers 200 with the schema's empty defaults, so a consumer that passes userId reads nothing and never learns why
metadata:
  type: reference
---

`GET /v1/configuration/<service>?userId=<id>` looks up the etcd key
`<env>/configuration/<service>/<userId>`. If that key does not exist there is
**no fallback to the global `<env>/configuration/<service>` key**:
`InternalGetConfiguration` (`src/use_case/internal_configuration.go`) fetches the
service's *schema* and returns `ChangeSchemaToConfig(schema)` — the schema's
default values — with **HTTP 200**.

So a consumer that passes a userId out of habit gets a successful response full
of empty defaults, with no error, no log line, and `version: 0` as the only
tell. Everything downstream looks like "the config is genuinely empty".

Measured 2026-09-11 on the same session, same endpoint:

```
?userId=28c87280-…  -> {"version": 0, "apps": []}
(no userId)          -> {"version": 2, "apps": [sellsuki, akita, patona, oc2plus, chat-console]}
```

This is what made the CCS3 app switcher render its empty state for every user on
every environment while migration `0003_seed_portal_app_registry_config` sat in
etcd unread — `portal-app-registry-rest.ts` was passing the profile's user id.
Fixed in CCS3 MR !512 by dropping the parameter.

**Rules of thumb**

- Pass `userId` ONLY when a per-user config for that namespace is actually
  seeded. For a global namespace, omit it.
- `version: 0` in a config response means "these are schema defaults", i.e. no
  config was found. Treat it as a miss, not as data.
- The global (userId-less) read is gated on
  `sellsuki.configsystem.config.view` over `sellsuki.user:""` — see
  [[reference_ccs_global_config_permission_gate]]. Omitting userId therefore
  turns a silent empty list into a 403 for users without that grant, which is
  louder and better, but the grant must be real per environment.

Related: [[reference_ccs_config_namespaces]] · [[reference_ambiguous_404_fail_open]]
(same bug class — a not-found path that returns success-shaped data).
