---
name: ai-chat-platform-plan
description: CTO ยกโจทย์เป็น AI Chat Assistant Platform 7 service (vendor WizeMoves ทำ insurance pilot FB Messenger 3 เดือน); แผนเต็มใน docs/ai-chat-assistant-platform-plan.md; BOLA chatbot → freeze สมอง + bridge
metadata: 
  node_type: memory
  type: project
  originSessionId: c107f1b5-8d84-48d1-b495-a8743dfcb35b
  modified: 2026-07-31T10:12:56.304Z
---

**AI Chat Assistant Platform (2026-07-26):** CTO แตกเป็น 7 service: (1) BOLA=LINE channel, (2) Messaging Service=FB+ช่องทางอื่น (revive `sellsuki-service-messaging` — Go, มี FB/LINE send repo, dormant ต.ค. 2023; ระวังชื่อชนกับ sellsuki-messaging-backend ตัว OTP/SMS), (3) Chat Core (repo `oc2plus-service-chat` NestJS8+Mongo dormant ก.พ. 2023 — เคาะ**สร้างใหม่เป็น Go** ใช้เดิมเป็น reference), (4) CRM=OC2Plus, (5) AI Agentic=Sellsuki RAG (มีจริง: `api-rag.dev-th.sellsuki.com/v1/api/chat/ask`, UI rag.staging — **ยังไม่ inventory repo = O1**) + AI adapter/MCP/multi-agent, (6) Data pipeline, (7) QMS=quota-management-backend (metering ตรง AI Gateway quota; hard cap ≠ feature gate)

**ที่มา:** vendor brief WizeMoves "AI Insurance Assistant" FB Messenger multi-tenant 3 เดือน (PDF ~/Downloads/Brief_Dev_AI_Chat_Assistant_Platform.pdf) — pilot ประกันภัย; AI Gateway + quota hard cap เดือน 3; code ownership = Sellsuki

**Tenancy เคาะแล้ว:** ไม่สร้าง multi-tenant core ใหม่ — tenant = **company ใต้ provider ใหม่ (insurance)** บน CCS/Kratos/Keto/invitation เดิม; ref_id=sellsuki.company:{id} pattern เดิม; config ต่อ tenant = CCS config namespace ใหม่; E0 = provider bootstrap + Keto role model + auth middleware contract ส่งให้ vendor (vendor ห้ามทำ auth เอง)

**Workspace layer เคาะแล้ว:** 3 ชั้น provider→company→**workspace** (zone ย่อยต่อ company แบบ BOLA) — workspace เป็น entity ของ chat platform เอง + Keto namespace `chat_workspace` (precedent BOLA-118 bola_workspace) ไม่ยัดเข้า CCS; channel binding/KB/persona/session/lead ผูก workspace; **scoping key หลัก = workspace_id ตั้งแต่ migration แรก** (ห้าม retrofit — บทเรียน 65 bugs BOLA); quota meter ต่อ workspace / hard cap enforce ต่อ company; auto-provision default workspace (progressive disclosure)

**แผนเต็ม:** `docs/ai-chat-assistant-platform-plan.md` — epic E0–E10, decisions D1–D6, open O1–O4. ความเสี่ยงหลัก = vendor ทำ monolith ซ้อน (ทางเลือก ข: vendor ทำ pilot-critical E1/E2/E7/E8 ใน boundary, ทีมในทำ E3/E4/E5/E6)

**ผลต่อของเดิม:** แผน 15 การ์ด BOLA personalized chatbot ([[bola-ai-chatbot-personalization]]) — requirement ย้ายบ้าน: B/K1/K2→E4 RAG กลาง, E/F1/F2→E2 user mapping, G/H/I→E3 agent service, J/OC-S→E6 CRM MCP; เหลือทำใน BOLA = A (encrypt key) + D (asker context) เป็น interim; BOLA chatbot = freeze สมอง + โหมด ai_backend=central ภายหลัง (E10); BOLA-294 ไม่กระทบ
