---
name: reference-push-after-mr-merged-orphans-commits
description: commit ที่ push ขึ้น branch หลัง MR ของ branch นั้น merge ไปแล้ว ค้างเงียบ ไม่มี MR พาไปไหน — ต้นเหตุ "แก้แล้วแต่หาไม่เจอ" ของ OC-4621
metadata:
  type: reference
---
GitLab MR ที่ merge แล้วไม่รับ commit ใหม่ commit ที่ push ตามเข้า source branch เดิมหลัง merge จะค้างอยู่ใน branch โดยไม่มี MR ไหนเห็น และไม่อยู่ทั้ง develop และ main
Title ของ MR เก่าอาจถูกแก้ให้ตรงกับงานใหม่ด้วย จึงอ่านแล้วเหมือนงานเข้าไปแล้ว

เกิดขึ้นกับ OC-4621 (backoffice !661): merge 20:13 พาไปแค่รอบแรกที่ทำทิศผิด ตัวที่ถูกต้อง push ตามไปตอน 21:56 ค้างอยู่ 4 วัน จนผู้ใช้ถามว่า "ทำหรือยัง หาไม่เจอ" สุดท้ายต้องเปิด !669 ใหม่

**How to apply:** เวลาตอบว่า "งาน X อยู่ไหน" ให้เช็ค `git merge-base --is-ancestor <sha> origin/develop` (และ main) กับ `glab mr view <n> -F json` ว่า state=merged หรือเปล่า อย่าเชื่อว่าเข้าแล้วเพียงเพราะชื่อ branch/MR ตรงกัน และก่อน push ขึ้น branch เดิม ให้ดูก่อนว่า MR ของมัน merge ไปแล้วหรือยัง
