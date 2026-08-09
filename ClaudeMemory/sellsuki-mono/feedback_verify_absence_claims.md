---
name: feedback_verify_absence_claims
description: "ก่อนเขียนว่า 'ไม่มี X' ในการ์ด/รายงาน ต้องกันเครื่องมือหลอกก่อน — clone refspec, ชื่อฟังก์ชันที่รีโปใช้จริง, rtk output"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-09T14:44:14.097Z
---

**ข้อกล่าวหาแบบ "ไม่มี X ในโค้ด" / "branch นี้ไม่มีจริง" มีต้นทุนสูงมาก** — มันไปลบเครดิตงานคนอื่นในการ์ด Jira · 2026-08-09 ผมเขียนลง OC-4398 ว่าการ์ดให้ข้อมูลเท็จ ทั้งที่**การ์ดถูกและผมผิด** ต้องไปถอนคอมเมนต์ต่อหน้าทีม

**Why:** negative claim พิสูจน์ไม่ได้ด้วยการหาไม่เจอ — เครื่องมือ "หาไม่เจอ" ได้หลายเหตุผลที่ไม่เกี่ยวกับความจริง

**How to apply — เช็ค 3 อย่างก่อนพูดคำว่า "ไม่มี":**

1. 🔴 **`git branch -r` เชื่อไม่ได้ถ้าไม่ได้ตรวจ refspec** — clone แบบ single-branch (`git config remote.origin.fetch` = `+refs/heads/main:...`) จะโชว์แค่ `origin/main` ตลอด ทำให้สรุปว่า "branch ไม่เคยถูก push" ทั้งที่รีโปมี ~90 branch
   ```bash
   git config --get remote.origin.fetch    # ต้องเป็น +refs/heads/*:refs/remotes/origin/*
   git config --unset-all remote.origin.fetch
   git config --add remote.origin.fetch '+refs/heads/*:refs/remotes/origin/*'
   git fetch origin
   ```
   เช็ค mainline ด้วย: `git rev-list --left-right --count origin/main...origin/develop` (OC2Plus หลายรีโป develop นำ main หลายร้อย commit)

2. **grep ด้วยชื่อที่รีโปนี้ใช้จริง ไม่ใช่ชื่อที่เราคาด** — หา `HasScope|RequireScope|CheckScope` ได้ 0 แต่ของจริงชื่อ `checkIdentityApiKey`/`isValidScopes` · วิธีกัน: หาจาก**จุดที่ต้องมี** (อ่าน use case ของ endpoint นั้นทั้งไฟล์) แทนที่จะหาจากชื่อที่เดา

3. **rtk hook คืน output ที่ถูกกรอง/ว่าง** — `git diff --name-status A...B` เคยคืนว่างทั้งที่มี diff จริง ทำให้เกือบสรุปว่า "branch merge ไปแล้ว" · ใช้ `/usr/bin/git` เมื่อคำตอบขึ้นกับความครบของ output (ดู [[reference_rtk_git_output_filtering]])

**นับ endpoint ให้ตรง surface** — "3rd-api มี 33 endpoints ที่ไม่มี scope" ผิด เพราะรวมเส้น customer-session เข้าไป · API-key surface จริงมี 4 · ก่อนสรุป coverage ให้แยก auth model ก่อนนับ

ดู [[project_oc2plus_3rdparty_apikey_gap]] · [[feedback_ground_claims_file_line]]
