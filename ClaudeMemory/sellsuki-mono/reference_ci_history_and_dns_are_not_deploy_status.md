---
name: reference_ci_history_and_dns_are_not_deploy_status
description: "อย่าสรุปว่า service ถูก deploy หรือยังจาก CI job history หรือ probe public DNS — ถาม kubectl; staging-th context มีอยู่แล้ว"
metadata:
  type: reference
---

**2026-09-18 — เขียนการ์ด AI-216 ว่า "3 ใน 5 service ไม่เคย deploy" แล้วผิดไป 2 ข้อ**

วิธีที่ใช้แล้วผิด:
- `glab api projects/:id/jobs?scope[]=success` แล้วหา `deploy` → "ไม่มี deploy ที่สำเร็จเลย"
- `curl https://<svc>.staging-th.sellsuki.com` → `000` → "ไม่ได้ deploy"

ความจริงที่ `kubectl` ตอบ:
- **rag-core รันมา 126 วันแล้ว** (+ `rag-core-python` อีก deployment บน arm64) — deploy เกิดนอกหน้าต่าง 100 job ล่าสุด
- `000` แปลว่า **ไม่มี public ingress** เท่านั้น — ทุก service ในนี้เป็น ClusterIP, การไม่มี DNS สาธารณะไม่ได้แปลว่าไม่มี pod
- `sellsuki-chat-core-common-secret` + DB `chat_core` **มีอยู่แล้ว** ทั้งที่การ์ดเขียนว่าต้องไปขอสร้าง — Emissary Host+Mapping ก็ถูกสร้างไว้ 20 วันแล้ว รอ backend

**Why:** CI history กับ public DNS เป็น proxy ของสถานะ deploy ไม่ใช่สถานะเอง และทั้งคู่ตอบ "ไม่" ได้ทั้งที่ของรันอยู่ — แล้วมันทำให้เปิดการ์ดสั่งงานที่ทำไปแล้ว ซึ่ง `shipping.md` ข้อ 11 บอกว่าแย่กว่าไม่มีการ์ด

**How to apply:** `kubectl` บน staging-th ใช้ได้เลย ไม่ต้อง Teleport ใหม่ (context `teleport.internal.staging-th.sellsuki.com-*` เป็น current อยู่แล้ว; production ก็มี context)
```bash
kubectl get pods -A | grep -i <service>          # มี pod ไหม (ทุก ns)
kubectl get svc -n sellsuki | grep -i <service>  # ชื่อ svc + port จริง
kubectl get secret -n sellsuki <name> -o go-template='{{range $k,$v := .data}}{{$k}} {{end}}'
kubectl get nodes -L kubernetes.io/arch          # arch ของคลัสเตอร์
```
พิสูจน์ในคลัสเตอร์ด้วย pod ชั่วคราว (`kubectl run --rm -i --restart=Never`) แทนการเดา — ใช้ `--overrides` ผูก `secretKeyRef` เป็น env เพื่อทดสอบ DB/endpoint **โดยไม่ต้องอ่านค่า secret ออกมา**

**ตัวเลขที่เดาแล้วผิดด้วย:** `RAG_CORE_BASE_URL` — อ่าน `containerPorts: 8001` จาก values แล้วจะเขียน `:8001` แต่ `rag-core-svc` เปิด **port 80** (Service map คนละเลขกับ container) ⇒ พอร์ตต้องอ่านจาก `kubectl get svc` ไม่ใช่จาก values

เกี่ยวกับ [[reference_ai_chat_has_no_staging_deployment]] (ต้องอัปเดต — rag-core ไม่ได้ขาด), [[feedback_verify_absence_claims]], [[reference_dev_th_cluster_access]]
