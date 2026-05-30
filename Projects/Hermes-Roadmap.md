# Hermes Roadmap

> สิ่งที่จะทำต่อใน Hermes — เรียงตาม priority และ effort
> อัปเดตล่าสุด: 2026-05-30 (batch /goal)

---

## ✅ เสร็จแล้ว (Done)

- [x] **Obsidian MCP** — เชื่อม `~/SecondBrain` เข้า Hermes (read + write)
- [x] **Git sync vault → GitHub** — auto-commit ทุก 15 นาที ไป `github.com/nuntnet/second-brain`
  - launchd: `ai.hermes.vault-sync.plist` (StartInterval 900s)
- [x] **True streaming** — char-based chunking (4 chars/45ms), เห็น typing จริง
- [x] **Per-session model** — แต่ละ chat เลือก model แยกกันได้
- [x] **Thinking indicator** — แสดง "Thinking… Xs" timer ระหว่างรอ
- [x] **Mobile layout** — RightPanel เป็น drawer, sidebar auto-collapse
- [x] **Memory Panel Tab 1** — SOUL.md editor (Personality)
- [x] **Memory Panel Tab 2 "Your Memory"** — query สิ่งที่จำ + add memory
- [x] **Thoughts → Hermes Sync** — ปุ่ม "Save to memory" + synced badge
- [x] **Sessions History Panel** — browse + view transcript + continue
- [x] **Scheduled Jobs Create UI** — form + presets + toggle + trigger + delete
- [x] **GitHub MCP** — repos/PRs/issues (token จาก gh keychain ไม่เก็บใน config)
  - ทดสอบแล้ว: Hermes ตอบ username + repo count ได้
  - wrapper: `~/.hermes/scripts/github-mcp.sh`
- [x] **Command Palette (Cmd+K)** — quick switch session/panel/model/recent chats
- [x] **Daily Briefing cron** — 7:00 GitHub + Obsidian tasks + top 3 priorities
- [x] **Memory Pipeline (End-of-Day cron)** — 21:00 extract decisions/tasks → Obsidian Daily
- [x] **Weekly Review cron** — ศุกร์ 17:00 → Obsidian Areas/Weekly-Reviews
- [x] **Mobile Access (cloudflared)** — launchd tunnel, URL ใน `~/.hermes/mobile-url.txt`
- [x] **Auth auto-recovery 2.0** — proactive token-keepalive (hourly) + แก้ wake-handler ไม่ restart adapter ทุก 5 นาที + adapter เรียก recovery เอง + `setup-long-lived-token.sh`
- [x] **Voice Interface (web UI)** — mic STT (Ctrl+B), TTS auto-speak toggle + per-message read-aloud, Thai voice (Kanya), Web Speech API (ไม่ต้อง backend)

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

## 🔒 Version Control (2026-05-30)

โค้ด Hermes แยกเป็น 2 private GitHub repo (ไม่เอาเข้า SecondBrain — กัน secret leak + vault bloat):

| Repo | Path | เนื้อหา |
|------|------|--------|
| `nuntnet/hermes-ui` | `~/hermes-ui` | Next.js web UI |
| `nuntnet/hermes-config` | `~/.hermes` | adapter, config.yaml, SOUL.md, scripts/ |

- **`~/.hermes/.gitignore`** ใช้ whitelist (ignore ทุกอย่าง → allow เฉพาะ code) — `auth.json`, `.env`, `*.db`, `logs/`, `memories/`, `sessions/`, `skills/` ไม่ขึ้น GitHub เด็ดขาด
- **SecondBrain vault** ยัง sync แยกที่ `nuntnet/second-brain` (notes เท่านั้น)
- ⚠️ **hermes-config commit แบบ manual** (ไม่ auto เหมือน vault) — แก้ config/scripts แล้วต้อง `cd ~/.hermes && git add -A && git commit && git push` เอง

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
