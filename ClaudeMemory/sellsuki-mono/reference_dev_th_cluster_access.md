---
name: reference_dev_th_cluster_access
description: วิธีเข้าไปทดสอบ service บน dev-th ด้วย kubectl + วิธีหา user ที่มีสิทธิ์จาก Keto แบบ reverse lookup
metadata:
  type: reference
---

**cluster:** `kubectl config` มี 2 context — production กับ `teleport.internal.staging-th...` · **dev-th อยู่บน cluster staging-th** ในคนละ namespace (เช่น `octoplus-dev` คู่กับ `octoplus` ที่เป็น staging, `share-dev` คู่กับ `share`)

**ทดสอบ service ที่อยู่หลัง gateway โดยไม่ต้อง login:** `kubectl port-forward -n <ns> svc/<svc> 1808x:80` แล้วยิงพร้อม header ที่ปกติ Oathkeeper ฉีดให้ (`X-User-Id`, `X-Company-Id`, ฯลฯ) — ได้ผลเหมือน gateway และแยกได้ว่าปัญหาอยู่ที่ gateway หรือที่ service · container port ของ Go service ที่นี่คือ **80 ไม่ใช่ 8080**

**หา user ที่มี permission จริงจาก Keto (reverse lookup 2 ขั้น)** — `kubectl port-forward -n share-dev svc/keto-read 14466:80`
1. `GET /relation-tuples?namespace=permissions&object=<permission.string>` → ได้ `relation` = tenant (`sellsuki.company:<uuid>`) และ `subject_set.object` = role (`sellsuki.role:<id>`)
2. `GET /relation-tuples?namespace=roles&object=sellsuki.role:<id>` → ได้ `subject_id` = `sellsuki.user:<uuid>`

ใช้ตรวจได้ด้วยว่า permission ใหม่ถูก provision จริงหรือยัง (จำนวน tuple = 0 แปลว่าโค้ด fail closed ทั้งระบบ)

**Kratos identity ของ dev:** `kubectl port-forward -n share-dev svc/kratos-admin 14434:80` → `GET /admin/identities`

⚠️ `curl ... | python3` โดน rtk hook กรองจนพังบ่อย — **เขียน response ลงไฟล์ก่อน แล้วค่อย parse จากไฟล์**

ดู [[reference_ccs_env_topology]] · [[reference_rtk_git_output_filtering]]
