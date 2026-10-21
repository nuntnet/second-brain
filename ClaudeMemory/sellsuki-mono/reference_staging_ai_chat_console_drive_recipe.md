---
name: reference_staging_ai_chat_console_drive_recipe
description: "วิธีเทส AI Chat console บน staging ใน pane เบราว์เซอร์ของ Claude — URL https://ai-chat-admin.staging-th.sellsuki.com, inbox ของ workspace อยู่ที่ /w/<workspaceId> (ไม่ใช่ /w/<id>/inbox → 404), login ผ่าน accounts.staging-th (PO ต้องล็อกอินใน pane เอง เพราะ cookie แยกจาก Chrome), ที่ desktop ≥1280px หัวห้องมี picker \"ยังไม่มีผู้รับ ▾ → รับเอง\" ใช้ claim ห้อง (D-008) และคิวมีปุ่ม \"รับเรื่อง\"; ที่ความกว้างแคบ control เหล่านี้หายไป · pilot workspace 5266e73a-c55d-434b-835c-17ce57507a08 (บริษัท Wizemovesทดลอง)"
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-21T17:22:56.492Z
---

ใช้จริง 2026-10-21 (AI-299 D-008 verify):
1. `navigate` ไป `https://ai-chat-admin.staging-th.sellsuki.com/` → ถ้าเด้งไป `accounts.staging-th.sellsuki.com/login` ให้บอก PO ล็อกอิน**ใน pane** (ล็อกอินใน Chrome ไม่ช่วย) แล้วอ่านหน้าซ้ำ — หน้าเลือกบริษัทจะขึ้น "CHAT CONSOLE เลือกบริษัท"
2. inbox: `/w/5266e73a-c55d-434b-835c-17ce57507a08` — มีแท็บ "ทั้งหมด N" / "คิวที่ยังไม่มีผู้รับ N" และ chip นับ ต้องการคน / AI ค้าง / AI กำลังคิด / แอดมินดูแล / AI ดูแลอยู่ / ปิดแล้ว (นับฝั่ง server, AI-299)
3. `resize_window 1280x900` ก่อน — ที่ 687px หัวห้องไม่มี picker/ปุ่ม claim (read_page interactive เห็นแค่ AI ตอบ/แอดมินตอบ)
4. claim: คลิก "ยังไม่มีผู้รับ ▾" ที่หัวห้อง → "รับเอง" → toast "มอบหมายสำเร็จ"; กลับแท็บ "ทั้งหมด" chip แอดมินดูแล +1 และแถวขึ้น "มอบหมายให้ <ชื่อ>" โดยโหมดยังเป็น AI ตอบ
5. เก็บหลักฐานจาก screenshot ใน tool-results ไปที่ `docs/cards/evidence/<CARD>/` แล้ว `resize_window preset desktop` ก่อนจบ

กับดัก: get_page_text หลัง navigate อาจยังเป็นหน้า 404 ของ SPA ก่อน redirect ไป login — อ่านซ้ำหนึ่งครั้ง · ห้องที่ claim ไว้จะค้างเป็นของ PO จริง (ไม่ใช่ fixture) บอกใน ledger/Jira
