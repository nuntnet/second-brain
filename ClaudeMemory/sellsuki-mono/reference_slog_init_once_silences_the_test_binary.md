---
name: reference_slog_init_once_silences_the_test_binary
description: sellsuki-go-logger v2 slog.Init มี sync.Once — ใครเรียกก่อนเป็นเจ้าของ logger ทั้ง test binary ทำให้ package-level init() ที่ปิดเสียง ทำให้เทสตัวอื่นในแพ็กเกจเดียวกันอ่าน log ไม่เจอ แล้ว fail เหมือนโค้ดพัง
metadata:
  node_type: memory
  type: reference
  originSessionId: 7b2f9d92-0c29-4b4e-bcb2-47995813fc48
  modified: 2026-10-13T03:12:09.976Z
---

`github.com/Sellsuki/sellsuki-go-logger/v2` — `slog.Init()` ห่อด้วย `sync.Once`
เรียกครั้งที่สองเป็น no-op เงียบ ๆ ไม่มี error ไม่มีทางรู้

ผลที่ตามมาในเทส: **เทสไฟล์หนึ่งตั้ง logger ให้ทั้ง binary** ถ้าไฟล์ไหนใส่

```go
func init() {
	slog.Init(sconfig.Config{LogLevel: level.Panic, Readable: true, ...})
}
```

เทสตัวอื่นในแพ็กเกจเดียวกันที่ **ยืนยันจาก log จริง** จะอ่านไม่เจออะไรเลย แล้ว fail
ด้วยข้อความที่อ่านเหมือนโค้ดโปรดักชันพัง (เจอจริง 2026-10-13: OC-4469
`TestAdjustMemberPointAuditLogSink` fail ที่ "one call = one audit line" ทั้งที่
audit + ledger event ถูก emit ถูกต้องทุกอย่าง — แม้แต่ `request_log` ก็หายไปด้วย
ซึ่งเป็นสัญญาณว่า logger เงียบ ไม่ใช่ handler ไม่ทำงาน)

**วิธีแยกแยะ**: ถ้า log หายไป *ทุกบรรทัด* รวมของ middleware ที่ไม่เกี่ยวกับโค้ดที่แก้
→ เป็นเรื่อง logger ไม่ใช่ feature ยืนยันด้วยการรันเทสตัวเดียวกันบน branch ที่ไม่มี
ไฟล์เทสตัวที่ตั้ง init() อยู่ด้วย

**วิธีแก้**: อย่าเรียก `slog.Init` จาก package-level `init()` ในไฟล์เทส — ย้ายไปเป็น
helper แล้วเรียกจาก constructor/helper ที่เฉพาะเทสกลุ่มนั้นใช้ เทสที่ fork ตัวเอง
(`exec.Command(os.Args[0], "-test.run=^X$")` เพื่ออ่าน stdout) จะได้ init ของตัวเองชนะ

บั๊กแบบนี้ **ไม่โผล่ตอนแต่ละ branch อยู่แยกกัน** โผล่ตอนเอามารวมเท่านั้น — เป็นเหตุผลหนึ่ง
ที่ควรรวม code+test ของการ์ดเดียวกันไว้ใน MR เดียวแล้วรันจริงก่อนเปิด

เกี่ยวข้อง: [[reference_robot_variables_are_case_insensitive_loop_var_overwrites_suite_var]]
(คนละภาษา กับดักเดียวกัน — ตัวแปร/สถานะ global ถูกทับเงียบ ๆ)
