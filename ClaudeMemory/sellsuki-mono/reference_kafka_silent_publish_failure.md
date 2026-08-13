---
name: reference_kafka_silent_publish_failure
description: BOLA staging pods have no KAFKA_SERVERS → every publish dies on localhost:9092 with only a Warn; and a topic full of messages is NOT evidence your producer works (105 were hand-seeded mocks)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
  modified: 2026-08-13T07:19:00.508Z
---

Verified on the staging pod 2026-08-13 (image `239251dc`, current main):

```
WARN use_case/follower.go:122  publish follower event failed
error: "kafka send keyed message failed: dial tcp [::1]:9092: connect: connection refused"
```

`back-office-of-line-api-backend` carries **no `KAFKA_*` env at all** — confirmed with
`kubectl exec … env` and an empty `envFrom`, not just by reading values files.
`KAFKA_SERVERS` was in `deployment/values-*.yml` once and was removed from all three
envs in commit `5b2b8cb "Update env"`. Fix requested in **BOLA-319**:
`KAFKA_SERVERS=kafka-cluster-kafka-bootstrap.datastore:9092` (that service exists in
ns `datastore`; catalog-service and oc2plus-line-crm already use that exact value).

Affects every BOLA producer, not just follower events: `bola.follower-events`,
`bola.contacts-import`, `bola.auto-push-message`.

## Why nothing surfaced

Four layers lined up:
1. env missing → `envDefault:"localhost:9092"` (`cmd/bola_server/main.go`)
2. `NewKafkaProducer()` **never returns nil**, so the `if uc.kafkaProducer != nil`
   guard always passes and it always attempts the send
3. the send error is logged `Warn` and **swallowed** (deliberate: must not block)
4. the call site is `go publishFollowerEvent(context.Background(), …)` — nobody was
   waiting for the result anyway

## The trap that cost a retraction

The topic **had 105 messages**, which looked like proof the producer worked. It
wasn't: all 105 were hand-seeded mocks in the pre-BOLA-265 schema
(`oc4209-mock-workspace-a`, `OC4209 QA Follower A`, `follower_id`/`occurred_at`,
**zero** with `follow_status`), keyed by `line_user_id` while the code keys by
`line_oa_id`.

Cheap discriminators, in order:
- **key vs payload** — real BOLA messages have `key == payload.line_oa_id`
- **a field only the new code emits** (`follow_status` here) — absent in every mock
- **the pod's own log** for the publish-failure line — decisive, one command

Lesson worth more than the config bug: *data existing downstream is not evidence the
upstream produced it.* Check the producer's own logs before concluding either way —
and do not reverse a verified claim because a screenshot suggests otherwise.

Related: [[reference_bola_staging_loki]], [[project_bola_deploy_values_in_repo]], [[feedback_verify_absence_claims]]
