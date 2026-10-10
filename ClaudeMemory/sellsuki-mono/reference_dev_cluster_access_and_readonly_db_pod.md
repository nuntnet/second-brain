---
name: reference_dev_cluster_access_and_readonly_db_pod
description: dev ของ OC2Plus/BOLA อยู่ใน cluster staging-th (ns octoplus-dev, bola-dev) — อ่าน log ได้ด้วย context ที่ระบุชื่อ, อ่าน DB ด้วย pod postgres ชั่วคราวที่ใช้ secretKeyRef ไม่แตะค่า secret
metadata:
  type: reference
---

(ตรวจ 2026-10-02) ไม่มี context ชื่อ "dev" — namespace `octoplus-dev` (member-api, backoffice-api ฯลฯ) และ `bola-dev` อยู่ใน cluster **staging-th**: ใช้ `--context teleport.internal.staging-th.sellsuki.com-teleport.internal.staging-th.sellsuki.com` ทุกครั้ง ห้ามพึ่ง current-context (เคยเด้งไป production เอง) · ต้อง `tsh` login ก่อน

- **log**: `kubectl --context $C -n octoplus-dev logs <member-api pod> --since=3h` — request_log มี method/path/status, body ของ register (มีเบอร์/อีเมลผู้ทดสอบ — อย่าบันทึกลงไฟล์โปรเจกต์, ลบไฟล์ชั่วคราวหลังใช้)
- **DB แบบอ่านอย่างเดียว**: `kubectl run tmp-readonly-… --restart=Never --image=postgres:16-alpine --overrides=<json>` โดย env ใช้ `valueFrom.secretKeyRef` (member-api: secret `oc2plus-crm-secret` keys `POSTGRES_HOST` / `POSTGRES_USERNAME` / `POSTGRES_PASSWORD` / `POSTGRES_CRM_DATABASE` (ชื่อเต็ม — `CRM_DATABASE` เฉย ๆ ทำให้ pod ค้าง CreateContainerConfigError, ตรวจ 2026-10-10) · DB ของ dev = `development_oc2plus_crm` (server เดียวกับ `staging_oc2plus_crm`) · ประวัติ migration อยู่ตาราง `migrations` ; BOLA: secret `bola-backoffice-secret` keys `DATABASE_HOST/PORT/USER/PASS/NAME/SSL`) แล้ว `psql -c "<select>"`; อ่านผลด้วย `kubectl logs` (อย่าใช้ `--rm -i` จะไม่ได้ output) แล้ว `kubectl delete pod` — ไม่เคยพิมพ์ค่า secret · ผู้ใช้อนุญาตเป็นกรณี ("ทำเองได้มั้ย") ไม่ใช่สิทธิ์ถาวร
- ตารางที่ใช้ได้: `oc2plus_bola_bindings`(company_id,slug,bola_workspace_id,binding_status), `member_app_slug_history`, `oc2plus_company_activation`; BOLA `liff_shell_destinations`(flow, destination_url) — คอลัมน์เป็น uuid ต้อง `::text` เวลาเทียบกับ varchar

**Why:** เจอว่า LINE login ล้มเพราะ slug ถูกเปลี่ยน (ดู [[feedback_check_the_norm_before_calling_it_broken]]) ต้องอ่านข้อมูลจริงถึงรู้ · ต่อยอด [[reference_bola_dev_db_not_readable_without_creds_or_client]]
