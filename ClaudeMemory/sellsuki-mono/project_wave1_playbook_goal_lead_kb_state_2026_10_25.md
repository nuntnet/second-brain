---
name: project_wave1_playbook_goal_lead_kb_state_2026_10_25
description: "2026-10-25 wave 1 ของ epic Playbook/Goal/Lead/KB (AI-218→316, 314, 315, 318, 325-327) โค้ดเสร็จและ push ทุก branch แต่ยังไม่มี MR และยังไม่มี verdict ของ verifier · ledger + หลักฐานอยู่ docs/cards/AI-*.md และ docs/cards/evidence/wave1-2026-10-25-*.md · ค้าง: kit tag v0.8.0, D-029..D-032, stack หลักสำหรับ verifier"
metadata:
  node_type: memory
  type: project
  originSessionId: 2103a08c-674d-4fb5-a849-24b47ce4bbe2
---

ผล wave 1 (สั่ง /feature 2026-10-25 ตี 2–5): worktree ใต้ `.worktrees/<CARD>-<repo>` branch ตัดจาก origin/main ต่อ repo (frontend/rag-core/messaging ใช้ sparse เพราะ sandbox เขียน `.claude/commands` กับ `.idea` ไม่ได้) · agent sonnet ต่อ card×repo เขียนเทสจาก AC แล้ว prove-red · ผู้ตรวจต่างโมเดล (opus) หา BLOCKER จริง 1 + SHOULD-FIX 5 ซึ่งแก้แล้ว.

**ค้าง (ต้องคนตัดสิน):** D-029 (AI-325 AC1-4 ต้องผู้ดูแล Vault, STOP) · D-030 (goal เก่าไม่เชิญจนกว่าบันทึกใหม่) · D-031 (prompted = ตัดสินใจเชิญ ไม่ใช่ส่งถึง) · D-032 (ตาราง company-shared นับเป็น workspace เดียวกัน + เปลี่ยนชื่อเมนูเป็น "ข้อมูลที่ต้องถามลูกค้า") · ตั้งเตือน Vault token ~2026-11-21/23 ที่ไหน · kit ต้อง merge+tag v0.8.0 ก่อน chat-core AI-316 build ผ่าน CI · verifier ยังไม่รัน (stack หลักมี branch อื่นรวมอยู่ และ docker/overmind ต่อไม่ได้จาก sandbox) · AI-318 precision/recall ยังไม่วัด (ต้อง LLM key).

**บทเรียน:** (1) อย่าให้ agent เทสด้วย body ที่คิดเอง — ปุ่ม provision ของ console ส่ง `{fields:[],expected_version:0}` ซึ่งการแก้ AI-218 รอบแรกตอบ 422 ทุกครั้ง (เทสผ่านหมด) ผู้ตรวจต่างโมเดลจับได้ → บรีฟต้องบอก "เปิด client จริงอ่าน request ที่มันส่ง" · (2) การแก้บั๊กแบบ "validate ก่อน provision" ต้องไล่ทุก branch ที่แตะเส้นเดียวกัน (AI-316 พาบั๊กเดิมกลับมาในที่ใหม่) · (3) migration ที่เพิ่มคอลัมน์ทำให้ pod เก่า error ไม่หาย — ดู [[reference_pgx_cached_plan_breaks_old_pods_after_add_column_migration]] · (4) agent บอก "ไม่มี Postgres" ทั้งที่ sandbox บล็อก — ดู [[reference_sandbox_blocks_localhost_so_db_tests_skip_silently]].

ที่เกี่ยวข้อง: [[project_playbook_goal_lead_kb_cards_2026_10_24]], [[feedback_tests_derive_from_the_card_not_the_code]]

**Later on 2026-10-25 (PO answers):** D-029 both ways (SRE card + reminders 14 Nov / 21 Nov; AI-325 due date 2026-11-14; `.ics` in docs/runbooks), D-030 A, D-031 **B built** (chat-core AI-314 `31d1c81`: slot reserved, confirmed after the reply is stored, retracted otherwise; full suite 6753 pass / 0 fail), D-032 3.1 A (3.2 unanswered), AI-315 term = "คำแนะนำจาก AI", word "เชิญ" → "ชวน" on the goal page (my pick), stack merge = A in two rounds but the PO gave no free time and the stack is RUNNING (chat-core, ai-agent, console, role-perm, kratos) so it has not been touched. Cards placed in AI Sprint 21–24 (3301–3304). New: AI-334 (contact `converted` content check, Sprint 22), AI-327 AC9 + chat-core `fix/AI-327-preview-unpaired-bold` `1a87313`.

**Evening 2026-10-25:** stack composed (chat-core, agent, console `local/stack` carry 218/314/315/318/327 tips; kit-dependent AI-316 and messaging AI-326 are not on it; DB migrated 0115+0116). Independent code review (31 agents) found 23 confirmed issues; 7 developer agents fixed what was clear (all on the card branches, pushed); open questions D-033..D-038 in the inbox. Independent verifiers (2 rounds) could confirm little on the real surface: Kratos 'dev' creds are the dev-th account (rejected locally), the seed identity `kratos` only writes to Test Insurance Co (85fa00dd), no leads/rooms with the needed data, flag source CCS :8085 not running. AI-318 AC1 measured: depends on the model (see [[project-ai318-interest-gate-depends-on-the-model-2026-10-25]]). Jira: sprints 3301-3304 hold the plan; AI-334 (contact converted check) created; AI-327 AC9 and AI-314 AC11 added; AI-333 got AC6 (panic leak).
