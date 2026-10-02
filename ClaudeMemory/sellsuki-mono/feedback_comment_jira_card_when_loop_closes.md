---
name: feedback_comment_jira_card_when_loop_closes
description: เมื่อการ์ดครบ loop (เปิด MR / merge develop / merge main / ปิดไม่ merge) ต้องคอมเมนต์ลง Jira + ย้ายสถานะให้ตรง MR — รูปแบบ repo / MR state / branch / commit + สิ่งที่ยังไม่ทำ
metadata:
  type: feedback
---

ทุกครั้งที่สถานะของการ์ดเปลี่ยน (เปิด MR, merge เข้า develop, merge เข้า main, ปิดโดยไม่ merge) ให้เขียนคอมเมนต์ที่การ์ดเลย ไม่ใช่รอตอนจบ และย้ายสถานะให้ตรงกับ MR (MR เปิด = In Review)

**Why:** 2026-10-03 การ์ด AI-272/273/275/277/278 ค้าง To Do ทั้งที่โค้ดอยู่ใน MR !162 ที่เปิดอยู่ — บอร์ดบอกว่า "ยังไม่เริ่ม" ทั้งที่งานเสร็จ ซึ่งเป็นทางที่งานถูกทำซ้ำ ผู้ใช้ขอให้ฝังเรื่องนี้ไว้ใน rules/skill

**How to apply:** กฎอยู่ที่ `.claude/rules/shipping.md` §15 (auto-load) · `/feature` ข้อ 18.5 · `/land` ขั้น 5 — รูปแบบ repository / MR (merged|pending-review|closed-no-merge + URL เต็ม) / branch / commit แล้วต่อด้วยสิ่งที่ยัง **ไม่** ทำ · อ้างเฉพาะสิ่งที่ตรวจต่อการ์ดจริง (git cherry ทั้ง branch ≠ หลักฐานต่อการ์ด — เคยพลาดในคอมเมนต์ AI-272) · เขียน Jira ทีละครั้งห่าง ~10 วินาที · อย่าย้ายการ์ดที่ตัวเองเขียน AC ไป Done

เกี่ยวข้อง: [[feedback_git_flow_main_develop_main]], [[reference_jira_mcp_writes_403_midsession]]
