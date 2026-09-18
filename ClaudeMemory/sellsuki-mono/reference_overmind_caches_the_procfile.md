---
name: reference_overmind_caches_the_procfile
description: overmind อ่าน Procfile ตอน daemon สตาร์ตครั้งเดียว — แก้ env ในไฟล์แล้ว restart ทีละ process จะไม่มีผลและไม่มีอะไรฟ้อง ต้องรีสตาร์ททั้ง stack
metadata: 
  node_type: memory
  type: reference
  originSessionId: 01c88554-c2ad-4379-b428-9806eeccfe92
  modified: 2026-09-18T16:33:14.066Z
---

overmind โหลด Procfile ตอน **daemon** สตาร์ต แล้วใช้สำเนานั้นตลอดอายุ session

⇒ เพิ่มตัวแปร env ในบรรทัดของ service แล้วสั่ง `overmind restart <proc>` process จะขึ้นมาใหม่ **โดยไม่มีตัวแปรนั้น** ไม่มี error ไม่มี log ฟ้อง อาการที่เห็นคือ "แก้ flag แล้วไม่ทำงาน" ซึ่งชวนให้ไปไล่หาบั๊กในโค้ดแทน

**ตรวจว่าตัวแปรถึง process จริงมั้ย** (macOS):

```bash
PID=$(lsof -nP -iTCP:<port> -sTCP:LISTEN | awk 'NR==2{print $2}')
ps eww -o command= -p "$PID" | tr ' ' '\n' | grep '^MY_VAR'
```

**วิธีที่ได้ผล** — รีสตาร์ททั้ง stack ของ Procfile นั้น เช่น ai-mvp:

```bash
make ai-mvp-stop && make dev-ai-mvp
```

เจอตอนเปิด `DURABLE_INBOUND_ENABLED` ใน `Procfile.ai-mvp` (2026-09-18) เสียไปหนึ่งรอบเต็ม

สองเรื่องที่พ่วงมาด้วย:

- `overmind start <proc>` **ไม่ใช่** คำสั่งสตาร์ตทีละ process — มันสตาร์ตทั้ง Procfile และจะบ่นว่า "Overmind is already running" ส่วนกฎในรีโปที่เขียนว่า `overmind stop <name>` แล้ว `overmind start <name> -D` ใช้ไม่ได้จริง ผลคือ process ค้างสถานะ `dead` ทางที่ใช้ได้คือ `overmind restart <proc>` (ตรวจ listener ซ้ำหลังทำ ว่าไม่มีตัวซ้ำบนพอร์ตเดิม)
- stack นี้มี socket หลายตัว (`.overmind-ai-mvp.sock`, `-messaging`, `-bola`, …) `overmind status` เปล่า ๆ อ่าน `.overmind.sock` จึงไม่เห็น chat-core เลย ต้องส่ง `OVERMIND_SOCKET` ให้ตรงตัว — ดู [[reference_overmind_dead_processes_are_a_wedged_session]]
