---
name: reference_oc2plus_backoffice_fe_local_against_branch_api
description: วิธีรัน backoffice FE worktree ยิงเข้า backoffice-api ของอีก branch ในเครื่อง + กับดัก prefix /v1 vs /backoffice/v1
metadata:
  type: reference
---

รัน **FE worktree + API worktree คู่กัน** เพื่อ verify ฟีเจอร์ end-to-end โดยไม่แตะ
server ที่ main checkout รันอยู่ (5176 / 8089)

```bash
# API worktree: .env ไม่ตามมากับ worktree ต้อง copy เอง (gitignored อยู่แล้ว ปลอดภัย)
cp backend/oc2plus-line-crm-service-backoffice-api/.env <api-worktree>/.env
# launch.json: PORT=8189 go run ./cmd/generics_server

# FE worktree: ลอก .env.development.local ของ main checkout แล้วชี้ target ไปพอร์ตใหม่
sed 's|^VITE_LOCAL_DEV_BACKOFFICE_TARGET=.*|VITE_LOCAL_DEV_BACKOFFICE_TARGET=http://localhost:8189|' \
  ../../.env.development.local > .env.development.local
ln -s ../../node_modules node_modules      # worktree ไม่มี node_modules
# launch.json: npx vite --mode development --port 5187
```

## 🔴 กับดัก prefix — เสียเวลาเพราะ error ชี้ผิดทาง

**backoffice-api mount ที่ `/v1` ไม่ใช่ `/backoffice/v1`**
(`fiber_server.go:69` `BaseURL: "/v1"`)

ฝั่ง FE เรียก `/backoffice/v1/...` ถูกแล้ว เพราะ **vite dev proxy ทำหน้าที่แทน oathkeeper**:
strip `/backoffice` ออก + ฉีด `X-User-Id` / `X-User-Kind` ให้ (`vite.config.ts:74-87`)

เวลา curl ทดสอบ API ตรง ๆ ต้องใช้ `/v1/...` — ถ้าใช้ `/backoffice/v1/...` จะได้
`404 Cannot GET` ซึ่ง**หน้าตาเหมือน route ยังไม่ถูก register** ทั้งที่ register แล้ว
· วิธีแยก: ยิง endpoint **ที่มีอยู่เดิม** ด้วย prefix เดียวกัน ถ้า 404 เหมือนกัน = prefix ผิด
ไม่ใช่โค้ดเรา · route ที่ register จริงจะตอบ **401** (ติดที่ auth) ไม่ใช่ 404

## permission ในเครื่องนี้

Keto ไม่มี tuple ของ company `11111111-1111-4111-8111-111111111111` เลย → permission ใหม่ทุกตัว
ได้ `is_allow:false` · หน้าที่ gate ด้วยสิทธิ์จะขึ้นโหมด read-only ซึ่ง**ถูกต้อง** ไม่ใช่บั๊ก
แต่แปลว่า write path ทดสอบผ่านเบราว์เซอร์ไม่ได้ถ้าไม่ grant ก่อน — ใช้ component test คุมแทน

ดู [[reference_oc2plus_local_stack_recovery_traps]] · [[project_oc3559_member_tier_state]]
