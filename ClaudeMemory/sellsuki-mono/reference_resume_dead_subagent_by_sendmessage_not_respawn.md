---
name: reference_resume_dead_subagent_by_sendmessage_not_respawn
description: subagent ที่ตายกลางคัน (API ENOTFOUND / เน็ตหลุด / org spend limit 429) ยังมี transcript อยู่ — SendMessage ไปที่ agentId เดิมจะ resume ต่อพร้อม context ทั้งหมด; การ spawn ใหม่ทำให้มันเริ่มอ่าน worktree ใหม่หมดและเสีย ~100k token · งานที่ commit แล้วใน worktree รอดเสมอ ส่วน uncommitted รอดถ้า worktree dir ยังอยู่ (scratchpad ถูกล้างเมื่อ session restart แต่ branch ใน .git ยังอยู่)
metadata:
  node_type: memory
  type: reference
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
  modified: 2026-10-17T09:23:55.768Z
---

เห็น 3 ครั้งใน session 2103a08c (AI-299, 2026-10-12 → 10-17):

- 10-12 กลางคืน: agent arena A/B ตายเพราะ **org monthly spend limit (429)** กลางงาน — worktree ใน scratchpad หายไปเมื่อ session restart (`git worktree list` ขึ้น `prunable`) แต่ **commit บน branch ยังอยู่ใน `.git` ของ submodule** (`git branch --list '*AI-299*'`) → เอา branch มาสร้าง worktree ใหม่ได้ สิ่งที่หายคือเฉพาะงาน uncommitted ของ agent → สั่ง agent ให้ commit เป็นหน่วยเล็ก ๆ ระหว่างทางเสมอ
- 10-17: FE developer agent ตาย 2 รอบด้วย `API Error: Can't reach the API server (ENOTFOUND)` — ไม่ใช่ความผิดของ agent; worktree + ไฟล์ dirty ยังอยู่ครบ
  - รอบแรกผม spawn ใหม่ (เสียเวลาให้มันอ่านงานเดิม) · รอบสองใช้ `SendMessage(to: <agentId เดิม>)` พร้อมข้อความ "resume: เน็ตหลุด เริ่มจาก git status/log ใน worktree นี้" → ทำต่อจนจบใน 1 รอบ
  - task-notification เขียนไว้เองว่า "The user can send it another message and resume it" — นั่นคือทางที่ถูก

**How to apply:** เมื่อ notification บอก `failed: Agent terminated early due to an API error` → `ToolSearch select:SendMessage` แล้วส่งข้อความ resume ไปที่ agentId เดิม ไม่ spawn ใหม่ · ใน brief ของ agent ทุกตัวสั่งให้ commit ระหว่างทาง และห้ามลบ worktree เอง · ก่อนประกาศว่างาน candidate "หาย" ให้ดู `git branch` ของ submodule ก่อน
