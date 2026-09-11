---
name: project-oc2plus-tier-is-per-company-config
description: PO เคาะ 2026-09-11 ว่า tier ทุกอย่าง (ชื่อ ตัวคูณ ส่วนลด ฐานวัด) เป็น config ต่อบริษัท design v3 เป็นแค่ตัวอย่าง config ไม่ใช่ spec
metadata:
  type: project
---

**PO เคาะ 2026-09-11:** ชื่อระดับ · ตัวคูณ · ส่วนลด · ฐานวัด → **configurable ต่อ company ทุกตัว** · **design v3 = ตัวอย่าง config ของบริษัทหนึ่ง ไม่ใช่ spec**

**Why:** แต่ละบริษัทออกแบบโปรแกรม loyalty ของตัวเอง ระบบต้องรองรับ ไม่ใช่ hardcode ชุดเดียว

**How to apply:** เลิกอ่านตัวเลขใน design (`BRONZE/300/1.2x`) หรือในการ์ด (`Silver/5,000`) เป็นข้อกำหนด — ทั้งคู่เป็นตัวอย่าง · การ์ดเจ้าของเรื่องคือ **OC-3559** ([[reference_oc2plus_jira_project]]) label `business` ไม่ใช่ `customer-app-program`

## ที่ต้องเพิ่มเข้า OC-3559

`member_tier` + `earn_multiplier` (default 1, ชั้นล่างสุดต้อง = 1) + `discount_percent` (default 0)
ระดับโปรแกรม: `qualify_basis` (enum) · `period_mode` + `period_length` (enum+int)

## 🔴 สองจุดที่ "configurable" ต้องมีขอบ ไม่งั้นกลายเป็นสอง engine

**1. ฐานวัด = คนละแหล่งข้อมูล** · แต้ม → `member_point_activity.quantity` (มีแล้วใน CRM) · **ยอดเงิน → ไม่มีใน point ledger เลย** ต้องดึงจาก `award_member_usage` ของ **3rdparty-api** = ข้ามบริการ · เสนอ enum ปิด `points_earned|spend_amount` และ v1 ทำแค่ตัวแรก

**2. หน้าต่างเวลา = เปลี่ยน engine ไม่ใช่เปลี่ยนค่า**

| | fixed_period (OC-3559 ออกแบบไว้) | rolling_window ("ยอดซื้อ 90 วัน" ใน design) |
| --- | --- | --- |
| `accumulated_points` | มีความหมาย รีเซ็ตตอนเลื่อนขั้น | **ไม่มีความหมาย** คำนวณจาก ledger ทุกครั้ง |
| `period_end` | มีจริง ใช้ sweep | **ไม่มี** |
| sweep | `WHERE period_end <= today` | ประเมิน**ทุกคนทุกวัน** |

schema S1 ของ OC-3559 เป็นของ fixed period ล้วน ๆ · เสนอ v1 ทำ `fixed_period` ให้จบ

## ตัวคูณกระทบ Award Engine (OC-4413 ยัง code review = จังหวะแก้ contract ถูกที่สุด)

ต้องเคาะ: **กฎปัดเศษ** (1.2x ของ 7 = 8.4) และปัดที่ขั้นไหน · **ซ้อนกับตัวคูณแคมเปญ** (คูณกัน vs ใช้มากสุด) · **audit ต้องเก็บ multiplier ที่ใช้จริง ณ ตอนนั้น** ห้ามอ่านใหม่ทีหลัง (สอดคล้อง L7 "ห้าม recompute") · แก้ตัวคูณแล้วห้ามย้อนคำนวณของเก่า

## ส่วนลด % ยังต้องเคาะแม้ configurable

ตอนซื้อของ (ต้องต่อ POS = นอกระบบเรา แค่ประกาศ) vs ตอนแลกของรางวัล (อยู่ในระบบ กระทบ OC-4354 ที่ต้องคิด `cost_point` ตามระดับ)

## ฝั่ง customer app

`ovBenefits` ต้อง **เรนเดอร์จาก config** ไม่ hardcode จำนวนชั้น/ชื่อ · **design ยังขาดวันหมดอายุของระดับ** ทั้งที่ JTBD ของ OC-3559 ต้องการ — ถ้าไม่แสดง สมาชิกตกระดับโดยไม่รู้ล่วงหน้า (และถ้าเลือก rolling_window จะไม่มีวันหมดอายุให้แสดงเลย)

เขียนไว้ที่ OC-3559 comment 44672 (เทียบ design vs การ์ด อยู่ที่ 44670)
