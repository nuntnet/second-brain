---
name: ai-backlog-gap-sweep-202608
description: "AI board gap sweep 2026-08 — สร้าง AI-142–150; E12 ปิดครบ 2026-08-23 (AI-178/179/180/181); AI-150 ติด decision A-vs-B ที่โค้ดพิสูจน์แล้ว"
metadata:
  node_type: memory
  type: project
  originSessionId: 22bc8e4d-cfe5-415c-8c23-d4d838fba867
  modified: 2026-08-23T13:45:00.000Z
---

Gap sweep บอร์ด AI (2026-08-14) ตอบคำถาม user "การ์ดขาดเยอะมั้ย โดยเฉพาะ admin UI":

- บอร์ดมี 141 ใบ ครอบเกือบหมด — §5.13.5 edits apply แล้วทุกใบ (verify ด้วย count-mode text query ตาม [[jira-mcp-search-quirks]])
- ปิดช่องแล้ว: **AI-147** Case&Checkpoint panel · **AI-148** Unmapped Fact Queue · **AI-149** Quota indicator/hard-cap banner · **AI-150** Workspace Members & Roles · E12: **AI-142** web chat adapter, **AI-143** Google OIDC bridge, **AI-144** reasoning-trace UI, **AI-145** Discord spike, **AI-146** template catalog+wizard
- รอบสอง (2026-08-15): **AI-151** spike ย้าย internal assistant · **AI-152** HR template v1 · **AI-153** unmapped-fact store · **AI-154** answer-trace persist · **AI-155** quota read model · **AI-156** case-switch UI · **AI-158** workspace-scoped invite (SP field = `customfield_10016`)
- รอบสาม (2026-08-18 /po-team): ชุด [E8][Observability] 6 ใบ — **AI-166/168/173/169/174/175** · decision log D1-D8 เป็น comment บน AI-9

## E12 ปิดครบ 2026-08-23 (4 ใบสุดท้าย)

**AI-178** (SP3) executable guard 4 ข้อ · **AI-179** (SP3) `case_type`/`case_id` ลง event envelope · **AI-180** (SP8) staff-as-asker · **AI-181** (SP8) ย้าย KB + ปลดระวาง channel-gateway — ทุกใบ parent = AI-131, มีผลตรวจโค้ดจริงฝังในการ์ด, สรุปรวมเป็น comment บน AI-131

**Why:** กันทำ gap analysis ซ้ำ กันสร้างการ์ดซ้ำ และผลตรวจด้านล่างคือของที่ต้องรู้ก่อนเริ่มงาน E12 ใบไหนก็ตาม

**ผลตรวจ guardrail §5.11 (2026-08-23, branch feature/AI-126-case-use-case):**
- 🔴 ข้อ 1 **ตก** — `src/entity/event/event.go:59-73` ไม่มี case_type/case_id เลย → AI-179
- 🟡 ข้อ 2 ผ่าน — `entity/lead/default_template.go:17-20` เป็น seed config; ref นอกไฟล์มีที่เดียว `entity/lead_reminder/default_config.go:54`
- 🟡 ข้อ 3 ผ่าน แต่ FE `packages/core/src/conversation/types.ts:42` มี `leadStage?` บน view model ห้องแชท (leak)
- ✅ ข้อ 4 ผ่าน — `src/guard/lead_boundary_test.go` แบนคำ funnel แต่ยังไม่แบน `lead`/`ticket` เป็น identifier

**กับดัก acid test:** `entity/case_/subject.go:10-31` SubjectType เป็น enum ปิด 2 ค่า + `case.go:87-88` บังคับ ChannelIdentityRef ⇒ staff asker ใส่ไม่ลง ⇒ เพิ่ม enum = แก้ core = ชน DoD ของ AI-131 · ai-agent รองรับ staff แล้ว (`agent/types.go:210-221`) · rag-core: AI-38 landed (`entity/workspace/workspace_scope.py`) แต่ KB internal ยังอยู่ `rag_internal` แบบ `filter_expr=None` (`collection_router.py:41-42`) · **ไม่มี pipeline ที่สร้าง KB เดิม** (ai-rag-inventory-recommendation.md:150-160)

## AI-150 — decision ที่โค้ดพิสูจน์แล้ว (ยังรอคนเคาะ)

CCS **รู้จัก** chat_workspace ระดับ company แล้ว (`central-control-backend/src/use_case/model/permission_group.go:18,217-222`) ⇒ assumption เดิมในการ์ดผิดบางส่วน · แต่ per-workspace operator **เป็นไปไม่ได้วันนี้**: `ai-platform-kit-go/authctx/roleservice.go:133-155` ยิง Kind=`chat_workspace` ซึ่งตกทุกครั้งเพราะ `entity@v0.22.0/entity.go:3-18` IsActor allowlist ไม่มีค่านี้ → InvalidArgument → false · โค้ดเขียนเองว่าเป็น "follow-up" ที่ยังไม่มีการ์ด

⇒ 2 ทาง: **A** เพิ่ม chat_workspace ลง allowlist ไลบรารีกลาง (ข้ามทีม) · **B** chat-core เก็บ workspace membership เอง (เสนอ B ตาม BOLA-309 layering) · **ข้อ "(ก) ทำได้ทันที" ในการ์ดใช้ไม่ได้** — ต้องมีการ์ด backend ก่อน

Related: [[sla-ladder-engine-state]], [[ai-mvp-integration]], [[bola-saas-access-model]]
