---
name: reference-oc2plus-member-c-vars-are-dead
description: ใน member frontend ตัวแปร CSS --c-* ทั้ง 233 จุดไม่มีใครตั้งค่าเลย ทุกจุดใช้ fallback ที่ฮาร์ดโค้ด ธีมต่อบริษัทจึงไม่ไปถึงหน้าหลัก
metadata:
  type: reference
---

`frontend/oc2plus-linecrm-frontend-member` — CSS ของ view อ่าน `var(--c-*)` **233 จุด ใน 16 ชื่อ** (`--c-primary` 69, `--c-muted` 34, `--c-border` 28, `--c-tint` 21, `--c-danger` 20, `--c-emerald` 13, `--c-fg` 12, `--c-font` 5, …) แต่ **ไม่มีอะไรตั้งค่าให้สักตัว** (ตรวจ 2026-09-11 บน develop @858c9a3)

| ตรวจอะไร | ผล |
| --- | --- |
| `--c-primary:` (หรือ `--c-*` ใด ๆ) ใน `src/` | ไม่มี |
| ใน `sellsuki-components/dist/style.css` (1.4 MB) | **0** |
| ใน bundle `.js` ของ DS | **0** |
| `setProperty(` ใน `src/` และใน DS bundle | **0 ทั้งสองที่** |

`applyTheme()` (`stores/theme/index.ts:57`) **ไม่เขียน CSS variable** — มันคืน object `Theme` ที่ถูกส่งเป็น prop ให้ `<ssk-theme-provider>` ซึ่งธีมได้เฉพาะ element `ssk-*`

**ผล: ทุก `var(--c-primary, #32a9ff)` ได้ `#32a9ff` เสมอ ไม่ว่าบริษัทตั้งสีอะไร** — กระทบ LoginView, MembershipView, PointClaim ทั้งสามหน้า, IdentityModal คือหน้าที่สมาชิกใช้จริงทั้งหมด · หลุดตอนที่หน้าเหล่านั้นถูกพอร์ตออกจาก `ssk-*` (ตอนนี้เหลือ `ssk-` 0 จุด)

อาการที่มองเห็น: `IdentityModal.vue` ใช้ fallback `#22c55e` (เขียว) ขณะที่ที่อื่นใช้ `#32a9ff` (ฟ้า) → สีแบรนด์ไม่ตรงกันโดยอุบัติเหตุ

**กระทบ [[project_oc2plus_member_app_design_v2]] และ OC-4246 (Theme Settings ที่สถานะ Done)** — แอดมินตั้งสีแล้วเปิดแอปไม่เห็นอะไรเปลี่ยนบนหน้าหลัก · เขียนไว้ที่ OC-4246 comment 44667

**เส้นทางที่ต่อกับธีมจริง** คือ token ชุดใหม่ของ OC-4493 (`--color-brand`, `--color-brand-dark`, `--font-family-base` ฯลฯ) ที่ `cssVars` bind ไว้บน `.app` ใน `App.vue` และมี unit test พิสูจน์ (`stores/theme/index.spec.ts`) · ตั้งใจใช้ชื่อใหม่ไม่ทับ `--c-*` เพราะ fallback เดิมบางตัวไม่ตรง palette

**บทเรียน:** เห็น `var(--x, fallback)` เยอะ ๆ อย่าเพิ่งเชื่อว่ามีใครตั้ง `--x` — เช็คว่ามี `--x:` หรือ `setProperty` จริงมั้ย ก่อนจะเคลมว่า theming ทำงาน
