---
name: reference_backoffice_fe_ci_skips_all_tests
description: "oc2plus-linecrm-frontend-backoffice ตั้ง UNIT/INTEGRATION/E2E_TEST_SCRIPT = echo \"1 + 1\" — pipeline เขียวไม่ได้แปลว่าเทสต์ผ่าน และ develop แดงอยู่ 181 เทสต์"
metadata:
  node_type: memory
  type: reference
---

`.gitlab-ci.yml` ของ `frontend/oc2plus-linecrm-frontend-backoffice` มีคอมเมนต์
`# Skip all test` แล้วตั้งค่าทับตัวแปรของ SRE template ทั้งสามชุด

```yaml
UNIT_TEST_SCRIPT:        'echo "1 + 1"'
INTEGRATION_TEST_SCRIPT: 'echo "1 + 1"'
E2E_TEST_SCRIPT:         'echo "1 + 1"'
```

⇒ job ชื่อ `unit_test_merge_request` **เขียวเสมอ** ไม่ว่าเทสต์จะพังแค่ไหน
**อย่าใช้ pipeline ของรีโปนี้เป็นหลักฐานว่าเทสต์ผ่าน** ต้องรัน `npx vitest run` เอง

ผลจริงเมื่อ 2026-09-18 (ชุดเดียวกัน เครื่องเดียวกัน):

| ref | ไฟล์แดง | เทสต์แดง | รวม |
|---|---|---|---|
| `origin/develop` เปล่า ๆ | 11 | **181** | 1029 |
| develop + !628 (OC-4089) | 10 | 170 | 1087 |

ไฟล์ที่แดงเรื้อรังบน develop: `AppSwitcher`, `services/bola/mock`,
`campaign/eligibility-channel-overlay`, `campaign/purchase-product/mock-service`,
`Bola/ContactList`, `Campaign/{List,MasterCreate,Preview,Rewards}`,
`MemberScreens/CustomerAppCard`

**วิธีตัดสินว่า branch ทำพังเพิ่มไหม** อย่าดูตัวเลขรวม ให้เทียบ *รายชื่อไฟล์* ที่แดง
ระหว่าง branch กับ develop เปล่า ๆ — ถ้าเป็น subset แปลว่าไม่มีของใหม่ (ตัวเลข
รวมเทียบกันไม่ได้เพราะ branch เพิ่มเทสต์เข้ามาด้วย)

worktree ต้อง `npm ci` ของตัวเอง (ไม่แชร์ node_modules) ~1 นาที

เชื่อม [[reference_oc2plus_backoffice_fe_red_on_develop]]
[[reference_green_count_hides_uncollected_suite]] [[reference_sre_test_job_slots_cannot_chain]]
