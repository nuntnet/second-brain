---
name: ai-placeholder-cards-review
description: 2026-08-31 po-team verdicts on AI-108/110/112/113/119 + open DEC list + E9 stranded-branch data-loss risk
metadata: 
  node_type: memory
  type: project
  originSessionId: 7a551d99-154b-4aa3-b896-e16e49d26658
  modified: 2026-08-31T16:40:10.441Z
---

รีวิว po-team 2026-08-31 — การ์ด AI 5 ใบที่ dev บอกว่า "develop ไม่ได้":

- **AI-108** → ควรปิด: Capacitor ถูกแทนด้วย Expo (D11), scope ซ้ำ AI-119+AI-66 ทั้งใบ; **epic AI-9 ยังเขียน Capacitor อยู่** ต้องแก้พร้อมกัน
- **AI-119** → แตก 4 slice: (A) PWA baseline **landed แล้ว** ใน `apps/admin/vite.config.ts:16-46` แค่รอ verify; (B) web push — **backend/ ไม่มี push infra ใดๆ** (ไม่มี VAPID/FCM/device-token table) ต้องสร้างใหม่ทั้งก้อน + admin ไม่มี per-conversation URL route (deep-link ได้แค่ inbox); (C) Kratos token mode — ขัด contract `server-verified-session/1.0.0` (AI-136/176 cookie-only) → park; (D) bundle 2.5MB
- **AI-113** → REQ-GAP: doc ขัดกันเอง (warehouse pipeline เป็นทางหลัก vs manual ใน pilot) ต้องถามทีม Data ว่า `facebook_ads_data_pipeline_v2` ครอบ pilot ไหม — ถ้าได้ควรปิดการ์ด; prod route ต่อ unavailableAdapter (ผู้ใช้เห็น error state)
- **AI-112 / AI-110** → placeholder ถูกแล้ว อย่าดึงเข้า sprint จนเคาะ DEC
- **DEC ค้าง:** DEC-1 ads pipeline (ทีม Data) · DEC-2 OMS-invoice vs Peak Engine (แนะนำ Peak) · DEC-3 O3 package↔Plan mapping ไม่มีเจ้าของ · DEC-4 token mode vs PAT-2587 · DEC-5 provider dashboard UI location · DEC-6 เจ้าของ domain "company จ่าย Sellsuki" (OMS ไม่มี invoice concept, SukiPay กลับทิศ)

⚠️ **Data-loss risk:** งาน E9 (AI-97/AI-98 ที่ทำให้ status ขึ้น In Review) อยู่บน branch local ที่**ไม่เคย push** (`feature/AI-97-etl-scaffold`, `feature/AI-98-dashboard-api`, `fix/e9-review-fixes`) และ repo เป้าหมาย `ai_chat_data_pipeline` (GitLab group `sellsuki/data-pipeline` นอก monorepo) **ไม่เคยถูกสร้าง** — clone ใหม่งานหายหมด

ต้นตอร่วม: `docs/ai-chat-assistant-platform-plan.md` + `docs/ai-platform-architecture.md` ไม่มี `Status:` header ตาม [[design-doc-authority]] และ decision ที่ถูกแทน (Capacitor, token mode) ไม่เคยถูก mark superseded

เกี่ยวข้อง: [[ai-admin-port-backend-map]] [[ai-board-stale-cards]] [[quota-no-allow-deny-rpc]]
