---
name: reference_envdefault_localhost_masks_missing_config
description: "A Go `envDefault` pointing at localhost turns a missing deployed env var into a connection error instead of a config error — the variable looks present everywhere, so the search goes to the wrong layer; delete the default rather than correcting it"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-09-17T16:03:17.167Z
---

**Found 2026-09-17 in `sellsuki-central-control-backend`, `cmd/generics_server/main.go`.**

```go
InvitationAPIBaseUrl string `env:"INVITATION_API_BASE_URL" envDefault:"http://localhost:8091"`
```

The variable was never set in the cluster. Because of the default, nothing ever
reported a missing config — the service happily dialled `localhost` inside a pod,
got a connection refused, and `assertInvitationAddressedTo` (which fails **closed**
by design) turned that into a 500 on every invitation accept.

**Three ways this misleads you, in order:**

1. **It is not a config error.** Config parsing succeeds. Nothing is nil. Every
   startup check passes.
2. **grep says the variable is "set".** `.env` sets it locally, so a local search
   confirms it exists, and the deployed gap stays invisible. I wasted a round
   proposing to change `8091` → `9999` — a port fix for a problem that was never
   about the port.
3. **The stack trace points at the callee.** The visible symptom is rps being
   unreachable, so the investigation goes to rps, which is fine.

**The fix is to delete the default, not correct it:**

```go
// no default — must be provisioned per env
InvitationAPIBaseUrl string `env:"INVITATION_API_BASE_URL"`
```

Then add the real value to `deployment/values-*.yml`
(see [[reference_ccs_deployment_values_live_in_the_repo]]) and log loudly at boot
when the URL is empty or loopback in a deployed environment.

**Generalise it:** an `envDefault` is only safe when the default is a *correct
production value*. A localhost/dev default on anything that crosses a service
boundary converts "you forgot to configure this" into "the network is broken",
which is a much more expensive question to answer. Same family as
[[reference_unset_kafka_topic_is_a_500]] and [[reference_ambiguous_404_fail_open]].
