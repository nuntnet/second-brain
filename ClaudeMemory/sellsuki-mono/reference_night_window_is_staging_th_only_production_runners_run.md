---
name: reference_night_window_is_staging_th_only_production_runners_run
description: "กฎ \"21:00–09:00 ไม่มีอะไรรัน\" ใช้กับ cluster staging-th (dev/staging apps + runner `Staging-th Runner General` / `base` / `ARM`) เท่านั้น — runner `Production Private Gitlab Runner` / `Production Gitlab Runner` (tag amd/docker/general) ยังรับ job ตอนกลางคืน (เห็น 2026-10-22 00:3x build ของ agent !88 รันบน Production Private) · job ที่ tag `staging-th` (deploy frontend, *_staging_th_*) ต้องรอเช้า แต่ MR test/build ทั่วไปอาจผ่านได้กลางคืนถ้า runner production หยิบ"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-21T17:22:37.323Z
---

สังเกต 2026-10-21/22 ระหว่าง /land:
- GitLab มี runner 5 ตัวที่โปรเจกต์เราเห็น: 5125 `Production Private Gitlab Runner` (tags general/docker/sellsuki/production/**amd**), 5127 `Production Gitlab Runner` (ไม่มี amd), 5153 `Staging-th Runner General` (amd, staging-th), 5154 `Staging-th ARM Runner` (arm), 5155 `Staging-th base runner` (staging-th)
- job tag `docker` อย่างเดียว → ลงได้ทั้ง 5; tag `amd+docker` (build image) → 5125 หรือ 5153; tag มี `staging-th` (deploy_staging ของ frontend, *_staging_th_*) → เฉพาะ 5153/5154/5155 = ปิดกลางคืนจริง
- ดังนั้น "MR เขียวตอนกลางคืน" เป็นไปได้ (2026-10-11 ก็เคยเห็น) และ "ค้าง pending ข้ามคืน" ก็เกิดกับ job ที่ไป staging-th เท่านั้น

**How to apply:** ตอนกลางคืนอ่าน `runner.description` ของ job ก่อนสรุป — job ที่ Production runner หยิบให้รอผลได้เลย ไม่ต้องรอเช้า; job ที่ติด staging-th ให้ปล่อยค้างแล้ว retry ทีละ id ตอนเช้า · เกี่ยวข้อง: [[feedback_morning_retries_in_waves_runner_cluster_is_12_nodes]]
