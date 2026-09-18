---
name: reference_fountain_registry_quota_full
description: "registry.fountain.sellsuki.com เต็มโควตา 300 GiB (2026-09-18) — build_image ตาย DENIED, deploy ถูก skip, merge แล้วแต่ของไม่ขึ้น dev"
metadata:
  node_type: memory
  type: reference
---

2026-09-18 ~09:00–10:09 ระหว่างวัน registry กลางเต็ม: job `build_image_*` คอมไพล์
ผ่านหมดแล้วมาตายตอน push

```
error pushing image: ... DENIED: adding 20.3 MiB of storage resource, which when
updated to current usage of 300.0 GiB will exceed the configured upper limit of 300.0 GiB.
```

**อาการที่หลอกคน** — stage `test` เขียวครบ (unit/integration/e2e) `build` แดง
แล้ว `deploy` ขึ้น **`skipped`** ไม่ใช่ failed ⇒ มองผ่าน ๆ เหมือน pipeline
"เกือบผ่าน" ทั้งที่ของไม่เคยขึ้น environment เลย · MR pipeline ก่อน merge เขียว
ได้ตามปกติเพราะไม่ push image ⇒ **pipeline ของ MR เขียว ไม่ได้แปลว่า deploy สำเร็จ**
ต้องดู pipeline ของ branch ปลายทางหลัง merge อีกรอบ

**retry ไม่ช่วย** จนกว่าจะมีคนล้าง image เก่า — เป็นโควตาของ registry ทั้งก้อน
ไม่ใช่ของ project · เทียบเวลาได้: backoffice-api pipeline 08:31 ยัง success,
member-api 10:09 แดง

วิธีเช็คว่าของขึ้นจริงไหม อย่าเชื่อ CI — ถาม cluster ตรง ๆ
```
kubectl -n octoplus-dev get deploy -o jsonpath='{range .items[*]}{.metadata.name}{"  "}{.spec.template.spec.containers[0].image}{"\n"}{end}'
```
tag ของ image = short sha ⇒ เทียบกับ `git rev-parse --short origin/develop` ได้เลย

เชื่อม [[reference_ci_history_and_dns_are_not_deploy_status]]
[[reference_pipeline_retry_runs_skipped_deploy_jobs]]
