---
name: reference_staging_morning_boot_restarts_on_keto_are_expected
description: "staging-th comes back from the night window around 09:00 ICT and Go services whose boot health check pings Keto gRPC (messaging-backend helper.go:537 'health check fail before start … ory_keto_grpc TRANSIENT_FAILURE') restart several times until Keto is up. A pod with RESTARTS 5-6 all between 09:01 and 09:10 is normal; 'RESTARTS 0 since yesterday' can never be a pass criterion on this cluster"
metadata:
  node_type: memory
  type: reference
  originSessionId: 4683d2cf-08fe-4531-bc03-8b12af31b5cb
  modified: 2026-10-28T06:24:28.144Z
---

Seen 2026-10-28 on `sellsuki-messaging-backend` (ns sellsuki): pod object created ~21:20 the night before (node drained), scheduled 02:01Z, container died 6× with exit 1 on the Keto gRPC boot check, stable from 02:10:22Z. `kubectl get pod -o json` → `.status.containerStatuses[0].lastState.terminated` + `kubectl logs --previous` tell the two apart from a chat-module or Vault failure in one call.

**How to apply:** when a morning checklist says "pods Ready, RESTARTS 0", read the restart timestamps before calling it a failure. Judge "no restart after the morning boot settled" instead. Any service with a `slog.Fatal` boot health check on Keto/Kratos/Vault/Kafka has the same pattern. Related: [[reference_night_window_is_staging_th_only_production_runners_run]], [[reference_messaging_chat_module_staging_flip_2026_10_27]].
