---
name: reference_teleport_session_kills_devth_access
description: When the teleport/VPN session expires, kubectl AND the dev-th public hostnames both die together while gitlab stays fine — recognise it before debugging the service
metadata:
  type: reference
---

อาการที่เจอ 2026-08-28: `kubectl` ตอบ `Unable to connect to the server: EOF` และ `curl https://crmapi.dev-th.oc2.plus/...` timeout **พร้อมกัน** ขณะที่ `gitlab.sellsuki.com` ยังตอบ 302 ปกติ และ DNS ของ dev-th ยัง resolve ได้

⇒ ไม่ใช่ service พังและไม่ใช่เน็ตหลุด — เป็น **teleport/VPN session หมดอายุ** (cert ของ teleport อายุสั้น) · `crmapi.dev-th.oc2.plus` ที่ดูเหมือน public จริง ๆ แล้วเข้าผ่านทางเดียวกัน

**วิธีแยกให้เร็ว:** ยิง gitlab เทียบ 1 ครั้ง — ถ้า gitlab ปกติแต่ dev-th timeout + kubectl EOF = session หมด ไม่ต้องไปไล่ pod/ingress
**แก้:** ให้ user รัน `tsh login` (เราทำแทนไม่ได้ ต้องมี MFA/รหัสของเขา)

⚠️ อย่ารายงานผลทดสอบ dev-th ว่า "ผ่าน/ไม่ผ่าน" ตอนนี้ — บอกตรง ๆ ว่ายังไม่ได้ยืนยัน แล้วเตรียมสคริปต์รันชุดเดียวไว้ให้รันทีเดียวตอน session กลับมา
