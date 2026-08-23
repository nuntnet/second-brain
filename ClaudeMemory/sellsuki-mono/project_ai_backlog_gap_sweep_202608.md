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

## AI-150 — ตัดสินแล้ว (เลือก A) + แตกการ์ด 2026-08-23

**หลักฐานที่พลิกคำตอบ:** rps รองรับ per-tenant role assignment อยู่แล้ว — `AssignRole(role_id, tenant, user, actor)` tenant อยู่บน **assignment ไม่ใช่บน role** · มี API ครบ (`ListUsersWithRoles`, `ListUserRolesInTenant`, `CountRoleMembers`, `UnassignRole`) · คำเชิญก็พา tenant ได้: `CreateInvitation.Assignments []*RoleWithContexts{RoleId, Contexts []*Namespace}` · CCS เป็น pass-through (`central-control-backend/src/repository/role_and_permission_repository/grpc.go:462-474`)

**🔑 precedent: `patona.store`** — `entity@v0.22.0/entity.go:8-10` มี `SellsukiCompany` + `PatonaStore` ใน IsActor allowlist ด้วยกัน (patona.store = tenant tier ใต้ company แบบเดียวกับ workspace เป๊ะ) และ CCS ทำ name-enrichment ให้มันแล้วที่ `src/use_case/invite.go:42-67`

⇒ **ตัวขวางคือสตริงเดียวใน allowlist** ไม่ใช่สถาปัตยกรรม ⇒ **เลือก A (ใช้ rps) ไม่ใช่ B (chat-core เก็บเอง)**

⚠️ ชื่อ tenant kind จะถูกแช่แข็งถาวร — วันนี้ kit ส่ง `"chat_workspace"` ซึ่งผิด convention `<product>.<thing>` → เสนอ `sellsuki.chat_workspace` · และ `IsActor()` ใช้ตรวจทั้ง Actor และ Tenant (ปนความหมาย แต่ patona.store ก็เป็นแบบเดียวกัน)

**การ์ดที่แตก (links wired):** **AI-182** (SP5, E0 — เปิด tenant tier + composite checker + port `list/grant/revoke` + `managedBy`) blocks **AI-150** (SP5 — เขียนใหม่เป็นหน้าจอ read + company-role, **ทำได้วันนี้** ผ่าน `ListUsersWithRoles` ที่ company tenant) blocks **AI-183** (SP5 — operator ราย workspace + เชิญ)

**ดีไซน์ที่ซื้อความยืดหยุ่น:** field `managedBy` (`ccs`|`workspace`) บนทุกแถว — UI แสดงครบทุกคนแต่กดแก้ได้เฉพาะที่ backend enforce ได้ · วันที่ allowlist ลง แถวเปลี่ยนค่าเอง ไม่ต้องแก้ UI/route (พิสูจน์ด้วย AC8 ของ AI-182: diff ต้องแตะแค่ `tenant.Kind`)

**AI-158 ต้อง rescope (SP ~8 → ~2):** premise ผิด — workspace-scoped invite **มีอยู่แล้ว**ทั้งสาย SPA→CCS→rps · เหลือแค่ enrichment ชื่อ workspace ข้าง ๆ `PatonaStore` และเป็นงานบอร์ด CCS · ถ้าปล่อยไว้จะกลายเป็นระบบคำเชิญที่สอง

Related: [[sla-ladder-engine-state]], [[ai-mvp-integration]], [[bola-saas-access-model]]
