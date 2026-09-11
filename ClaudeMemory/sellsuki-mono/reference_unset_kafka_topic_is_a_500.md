---
name: reference-unset-kafka-topic-is-a-500
description: "An unset KAFKA_TOPIC_* in OC2Plus backoffice-api is a 500 cannot_publish_message, not an inert feature flag — setup.sh wrote only 2 of the 6"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f1e1e4b4-53e4-4459-99e1-ea7fc29faaa9
  modified: 2026-09-11T11:10:06.414Z
---

`backend/oc2plus-line-crm-service-backoffice-api` builds a kafka-go writer per
topic at boot (`cmd/generics_server/helper.go:307-326`), from six env vars
declared in `cmd/generics_server/main.go:71-77`. `scripts/setup.sh` wrote only
`KAFKA_TOPIC_CAMPAIGN_STATUS_COMMAND` and
`KAFKA_TOPIC_POINT_CLAIM_NOTIFICATION_EVENT`. The other four defaulted to `""`,
the writer was still constructed, and every publish failed — so
**POST /code_lot** and **campaign save** answered
`500 {"error_code":"cannot_publish_message"}` on any fresh local stack.

**Why it misleads:** it looks exactly like a missing migration or a stale
checkout, and both were checked first and were fine. `KAFKA_ENABLED=false` does
not gate any of this. Read the response body out of the http_response log line
before theorising — the error_code names the layer.

Fixed 2026-09-11: all six now in `scripts/setup.sh`'s template, and
`scripts/seed-dev.sh` creates them (it previously created six topics, none of
them OC2Plus).

**Local limit that remains:** these four are producer-only here; the consumers
live in a worker outside this workspace. So a code lot publishes and then sits
in `generating` forever, and an export never produces a file. That is expected
locally, not a new bug.

See also [[reference_kafka_silent_publish_failure]],
[[reference_stray_claude_dev_server_squats_port]].
