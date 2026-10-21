---
name: feedback_morning_retries_in_waves_runner_cluster_is_12_nodes
description: "2026-10-21 — retry job ที่ตายตอนกลางคืน \"ทีละ id\" ไม่ได้แปลว่ายิงทั้งหมดพร้อมกัน; 28 retry ใน 2 นาทีทำ shared runner (0/12 nodes) เต็ม → รอบสองตายเป็น runner_system_failure \"Insufficient cpu/memory\" และ deploy staging ที่ replay ก็ตายด้วย · ต้อง retry เป็นระลอก (≤6–8 job ต่อรอบ, backend/deploy ก่อน) แล้วอ่านผลครั้งเดียวก่อนยิงรอบถัดไป · auto-mode classifier ก็ปฏิเสธ loop retry ที่ยิงรวดเดียว ([Blind Apply]) ให้แยกต่อ MR"
metadata:
  node_type: memory
  type: feedback
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-21T02:37:36.451Z
---

เช้าวันที่ 21 ต.ค. 2026 (/land หลังคืนที่เปิด MR 11 ตัว): ผมไล่ `POST jobs/:id/retry` ให้ทุก job ที่ตายตอนกลางคืนรวม ~28 job ภายใน 2 นาที (chat-core 4 MR + main pipeline, admin-fe, agent, management-backend) + replay deploy staging 2 ตัว

ผล: runner cluster ของ GitLab เป็น k8s แชร์กัน 12 node (3 เป็น arm, 4 datastore, เหลือไม่กี่ node ให้ runner) → pod ของ job ใหม่ Pending ด้วย "0/12 nodes are available: 1 Insufficient memory, 4 Insufficient cpu…" จนหมดเวลา = `runner_system_failure` รอบสอง ซึ่งดูเหมือน "แดงซ้ำ ไม่ใช่ flake" ทั้งที่สาเหตุคือคิวที่ผมสร้างเอง และ deploy ที่ replay (admin-fe 272636, Control Center 272637) ก็ตายแบบเดียวกัน

**Why:** กฎ "retry ทีละ job by id ครั้งเดียว" (shipping.md §9) คุมเรื่อง *ไม่ retry ทั้ง pipeline* แต่ไม่ได้บอกเรื่องปริมาณ; cluster มีที่ให้ job พร้อมกันราว 6–8 ตัว

**How to apply:**
- จัดลำดับก่อน retry: deploy ที่ค้างจากเมื่อคืน + main pipeline ก่อน → backend MR → frontend MR; ยิงรอบละไม่เกิน ~6–8 job แล้วอ่านสถานะ**ครั้งเดียว** (ไม่ poll) ก่อนยิงรอบถัดไป
- เจอ `runner_system_failure` รอบสอง ให้อ่าน log ก่อนจัดประเภท: "Unschedulable … Insufficient cpu/memory" = คิวเต็ม (รอ drain แล้วยิงใหม่ได้ ไม่นับเป็น retry รอบสาม) ส่วน "Node is not ready"/`started_at = null` = ช่วงปิด
- auto-mode classifier ปฏิเสธ loop `for … retry` ที่รวมทุก project ([Blind Apply]) แต่ยอมเมื่อแยกเป็นคำสั่งต่อ MR/ต่อ pipeline — แยกตั้งแต่แรก
- `dependency_scan_govulncheck` แดงพร้อมกันทุก MR ของ repo = stdlib เลื่อน (GO-2026-6809 → go1.26.10) ไม่ใช่ diff; แก้ด้วย `refactor/` branch bump toolchain 3 จุด (go.mod toolchain, CI GOTOOLCHAIN, Dockerfile ENV GOTOOLCHAIN — ตัวหลังค้างที่ 1.26.6) แล้ว merge main เข้า MR ที่เปิดค้าง · เกี่ยวข้อง: [[reference_sandbox_blocks_go_cli_tls_and_ssh]] (toolchain download ต้องรันนอก sandbox)
