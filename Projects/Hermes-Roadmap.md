# Hermes Roadmap

> สิ่งที่จะทำต่อใน Hermes — เรียงตาม priority และ effort
> อัปเดตล่าสุด: 2026-05-30

---

## ✅ เสร็จแล้ว (Done)

- [x] **Obsidian MCP** — เชื่อม `~/SecondBrain` เข้า Hermes (read + write)
- [x] **Git sync vault → GitHub** — auto-commit ทุก 15 นาที ไป `github.com/nuntnet/second-brain`
  - คำถาม "ต่อ GitHub เพื่อ backup Obsidian ใช่มั้ย?" → **ใช่ และทำเสร็จแล้ว**
  - launchd: `ai.hermes.vault-sync.plist` (StartInterval 900s)
  - script: `~/.hermes/scripts/sync-vault.sh`
- [x] **True streaming** — char-based chunking (4 chars/45ms), เห็น typing จริง
- [x] **Per-session model** — แต่ละ chat เลือก model แยกกันได้
- [x] **Thinking indicator** — แสดง "Thinking… Xs" timer ระหว่างรอ
- [x] **Mobile layout** — RightPanel เป็น drawer, sidebar auto-collapse
- [x] **Memory Panel Tab 1** — SOUL.md editor (Personality)
- [x] **Auth auto-recovery** — wake-handler เปิด Claude.app refresh token

---

## 🔥 Priority 1 — UI Features ที่ใกล้เสร็จ (ต่อจากของเดิม)

- [ ] **Memory Panel Tab 2 "Your Memory"**
  - ปุ่ม "Ask Hermes what it remembers" → query facts ที่ Hermes จำได้
  - แสดงเป็น bullet list
  - ปุ่ม "Add memory" → textarea ส่ง "Please remember: …"
  - _effort: 1 ชม._

- [ ] **Thoughts → Hermes Memory Sync**
  - ปุ่ม "Save to Memory" บน thought card
  - ส่ง "Please remember this: {content}" → mark `synced: true`
  - badge "✓ saved to memory"
  - _effort: 30 นาที_

- [ ] **Sessions History Panel**
  - browse session เก่า (title, date, message count)
  - กด session → ดู transcript ใน modal
  - ปุ่ม "Continue this session"
  - API route: `/api/hermes/sessions`
  - _effort: 1 ชม._

- [ ] **Scheduled Jobs Create UI**
  - form สร้าง cron job (name, schedule, prompt, enabled)
  - cron picker + human-readable preview
  - POST `/api/hermes/cron/jobs`
  - _effort: 30 นาที_

---

## 🚀 Priority 2 — Automation & Intelligence

- [ ] **Memory Pipeline (auto-capture)**
  - หลัง chat จบ → auto-extract facts → เขียนลง Obsidian
  - flush memories ผ่าน Hermes memory provider
  - เลือก folder ปลายทาง (Daily/ หรือ Areas/)
  - _effort: 2-3 ชม._

- [ ] **Proactive Daily Briefing**
  - cron 7:00 น. → morning summary
  - รวม: GitHub PRs ค้าง, Obsidian tasks, context เมื่อวาน
  - ส่งผ่าน Discord / Telegram
  - _effort: 1 ชม. (ต้องมี GitHub MCP ก่อน)_

---

## 🔌 Priority 3 — MCP Connections (เพิ่มความสามารถ)

- [ ] **Brave Search MCP**
  - ให้ Hermes ค้นเว็บได้
  - `npx @modelcontextprotocol/server-brave-search` + API key
  - _effort: 15 นาที_

- [ ] **GitHub MCP**
  - Hermes รู้จัก repos, PRs, issues
  - `npx @modelcontextprotocol/server-github` + PAT
  - ปลดล็อค daily briefing + code workflow
  - _effort: 15 นาที_

- [ ] **Google Calendar MCP**
  - อ่าน/สร้าง event
  - daily briefing รวมตารางนัด
  - _effort: 30 นาที (OAuth setup)_

- [ ] **Gmail / Email MCP**
  - อ่าน inbox, draft reply
  - _effort: 30 นาที (OAuth setup)_

---

## 📱 Priority 4 — Mobile & UX

- [ ] **Mobile Access (cloudflared tunnel)**
  - เข้า Hermes UI จาก Samsung ผ่าน browser
  - `cloudflared tunnel` → public HTTPS URL
  - หรือใช้ Telegram bot แทน (เบากว่า)
  - _effort: 1 ชม._

- [ ] **Command Palette (Cmd+K)**
  - quick switch session / panel / model
  - quick action: new chat, search, scope folder
  - _effort: 2 ชม._

- [ ] **Voice Interface**
  - Ctrl+B record → STT (Whisper local) → ส่ง
  - response → TTS (edge voice)
  - config มีอยู่แล้วใน config.yaml (`voice.record_key: ctrl+b`)
  - _effort: 2-3 ชม._

---

## 📝 Notes / Decisions

- **Obsidian path** = `~/SecondBrain` (ไม่ใช่ ~/Documents/SecondBrain)
- **Model default** = `cc/sonnet` (เร็ว, เห็น streaming ชัด) — Opus ช้า 20+ วิ
- **RAM**: Docker เป็นต้นเหตุ service ตาย — ปิด AutoStart ไว้
- **Telegram > iMessage** เพราะใช้ Samsung (Apple Skills ใช้กับมือถือไม่ได้)
- Extended thinking (Show thinking) จะเห็นเฉพาะตอนใช้ Opus ที่ทำ internal CoT — SDK ไม่ให้ config budget เอง

---

## ลำดับแนะนำ (next session)

1. **GitHub MCP + Brave Search** (30 นาที รวม) → ปลดล็อคเยอะสุด
2. **Memory Panel Tab 2 + Thoughts Sync** (1.5 ชม.) → ปิด loop memory
3. **Daily Briefing cron** (1 ชม.) → เห็นประโยชน์ทุกเช้า
4. **Sessions Panel + Scheduled Jobs UI** (1.5 ชม.) → QoL
