---
name: project_central_audit_log
description: Central user-activity audit log epic PAT-2611 — stdout(sellsuki-go-logger v2)→fluentd→Loki→CCS-backend query module→CCS1/2/3 UI; แยกจาก SukiPay transaction audit
metadata: 
  node_type: memory
  type: project
  originSessionId: 80b5b310-7a4e-49dc-8932-34d4eb091c9a
  modified: 2026-08-11T07:09:27.371Z
---

**Epic PAT-2611** = central user-activity audit log (ใครทำอะไรทั่วแพลตฟอร์ม) — สร้าง 2026-08-07 ตาม vision user. Stories: PAT-2612 (S1 standard+adoption), PAT-2613 (S2 fluentd→Loki), PAT-2614 (S3 query module ใน sellsuki-central-control-backend), PAT-2615 (S4 UI ที่ CCS1/2/3), PAT-2616 (S5 retention/archive). ทั้งหมดลง project PAT.

**สถาปัตยกรรม:** ทุก service emit `slog.Audit` → stdout (log_type=audit) → fluentd → Loki audit stream → query module ใน **sellsuki-central-control-backend** (ไม่ใช่ service ใหม่ — user เคาะ) → UI ที่ CCS1/2/3. Query module บังคับ tenant scope ฝั่ง server ผ่าน `entity_owner_type/id`.

**Audit format มาตรฐาน (มีอยู่แล้ว):** `github.com/Sellsuki/sellsuki-go-logger/v2` struct `log.AuditPayload` (log/audit.go:5-20): actor_type, actor_id, action (enum create/update/delete/access), entity, entity_refs[], entity_owner_type, entity_owner_id. เรียก `slog.Audit(fn, log.AuditPayload{...})`. **18 service ใช้ v2.1.1 แล้ว ยกเว้น kratos-ui-go + pis-api ยัง v1.1.1 (เก่า ต้อง upgrade — S1)**.

**CCS = Central Control System 3 ระดับ** (business-context.md): CCS1=System (`sellsuki-system-management-frontend` :5179, เห็นหมด) · CCS2=Provider (`sellsuki-provider-management-frontend` :5178) · CCS3=Company (`sellsuki-company-management-frontend` :5177). ทั้ง 3 แชร์ backend `sellsuki-central-control-backend`.

**แยกจาก SukiPay:** central audit นี้ ≠ SukiPay transaction audit ([[project_sukipay_offline_payment]] / PAT-2049 DB table 7yr PAT-2125) — คนละ data/purpose. Loki infra อาจ share ([[reference_bola_staging_loki]] system-logging ns). Related เดิม: PAT-273 (logger standard), PAT-161 (generic CCS audit viewer ที่ S3+S4 realize — link/ปิดเมื่อเสร็จ), PAT-2383 (ccs3 sukipay audit table = UI precedent). Jira type "Tech Story" create ไม่ผ่าน → ใช้ "Story".
