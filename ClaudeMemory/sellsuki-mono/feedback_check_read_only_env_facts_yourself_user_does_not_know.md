---
name: feedback-check-read-only-env-facts-yourself-user-does-not-know
description: "When blocked on a read-only fact about dev/staging (is permission X migrated, does role Y carry it), check it yourself — teleport is logged in and a temporary port-forward + grpcurl answers it; asking the user fails because they do not know. 2026-10-11"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 5077123b-d938-4a36-b54b-335561e7b9a9
  modified: 2026-10-11T14:17:28.566Z
---

On 2026-10-11 I asked the user "does dev have `oc2plus.consent.manage`? checking needs reading the cluster, I ask for a go each time". The user answered: "คุณเช็คได้มั้ย … ผมไม่รู้หรอก" — they could not answer the question and wanted me to look.

**Why:** the fact was read-only and cheap to get: `tsh status` already logged in as k8s-cluster-admin, rps lives in ns `sellsuki-dev` of the staging-th cluster (context `teleport.internal.staging-th.sellsuki.com-teleport.internal.staging-th.sellsuki.com`), `kubectl port-forward <rps pod> 15051:50051` then `grpcurl -plaintext -d '{"code":"…"}' localhost:15051 role_and_permission.RoleAndPermissionService/GetPermissionByCode` and `GetRole {"role_id":N}` (dev test company fe621d66… owner role is 89586). Stop the port-forward afterwards, delete temp files, never print secrets. Result that day: permission exists, role 89586 carries it.

**How to apply:** a "needs human to look at the cluster" blocker is mine to clear when it is a GET: do it, record the check in the card ledger's shipping gate with what was NOT checked (other companies, Keto tuples). Writes (CreatePermission, MigrateRoles, UpdateRole on a system role) are still blocked in auto mode and are the user's. Related: [[reference-dev-cluster-access-and-readonly-db-pod]], [[reference-dev-rps-membertier-missing-0025-present]], [[feedback-decisive-deep-execution]].
