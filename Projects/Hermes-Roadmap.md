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
- [x] **Voice Interface (web UI)** — mic STT (Ctrl+B), TTS auto-speak + per-message read-aloud
- [x] **Thai quality fix** — SOUL.md persona (ผม/ครับ สม่ำเสมอ) + adapter ไม่ route ไทย/CJK ไป Haiku (ขั้นต่ำ Sonnet) + honor explicit model
- [x] **TTS quality fix** — edge-tts (th-TH-NiwatNeural), strip emoji/symbols, ไทยปนอังกฤษใช้ multilingual voice (ไม่สะกดทีละตัว)
- [x] **Learning loop** — memory write/recall verified ✓; **Memory Distill cron** (22:00) consolidate+dedupe USER.md (69KB→5KB), เก็บ Patterns + Avoid/Mistakes sections; 👍/👎 feedback → feedback.jsonl
- [x] **Telegram bot** — long-polling, lock to user, cron delivery → มือถือ Samsung (setup-telegram.sh)
- [x] **Sonnet default** — opus-4-5 → sonnet-4-6 (Telegram/CLI เร็วขึ้น; UI เลือก model เองได้)
- [x] **Streaming TTS** — พูดทีละประโยคระหว่าง stream (queue) เริ่มพูด ~1s ไม่รอจบ; sentence split ฉลาด (Thai/list/version)
- [x] **hermes-backup.sh + cron** — commit+push 2 repo คำสั่งเดียว, launchd ทุก 6 ชม. (commit เฉพาะมี changes)
- [x] **TTS segmentation** — แยกภาษา ไทย→Niwat อังกฤษ→Andrew ต่อเสียง (แก้อังกฤษ "หายในลำคอ")

---

## 🩹 Stability fixes (2026-05-30)

- [x] **Outline MCP** — Hermes ค้น docs.sellsuki.com ได้ (28 collections) — key อ่าน runtime
- [x] **Morning Briefing → standup+PR digest** — GitHub PR review + Obsidian tasks + top 3 (ส่ง Telegram)
- [x] **🔥 แก้ MCP process leak (ต้นเหตุ RAM crash!)** — obsidian-mcp leak 11 procs/573MB → 2/65MB
  - keepalive patch: stdio ข้าม probe (ดู scripts/PATCHES.md — re-apply ถ้า update hermes)
  - mcp-reaper watchdog: ฆ่า orphan ppid=1 ทุก 10 นาที
- [ ] **ปิด terminal hermes เก่า (PID 3284)** — เปิดค้าง 6 วัน รัน code เก่า ⚠️ ผู้ใช้ต้องปิด/restart เอง

---

## 🎯 ที่เหลือจริงๆ (เกือบทุกอย่างเดิมเสร็จหมดแล้ว — ดู Done ด้านบน)

### 🔌 Integration — เชื่อมโลกการทำงาน (คุ้มสุด)
- [ ] **Outline MCP เปิดให้ Hermes** — มีใน ~/.claude/mcp.json แล้ว แค่เพิ่มใน config.yaml → ถาม docs.sellsuki.com ได้ (_15 นาที, ไม่ต้อง credential_)
- [ ] **Jira / Atlassian MCP** — อ่าน/อัปเดต ticket, สรุป sprint (_มี MCP พร้อม, ต้อง auth_)
- [ ] **Google Calendar MCP** — อ่าน/สร้าง event + รวมใน briefing (_OAuth setup_)
- [ ] **Gmail MCP** — อ่าน inbox + draft reply (_OAuth, ใช้ร่วม Calendar_)
- [ ] **Index codebase** — semantic search ch-erawan-next / OMS 2.0

### 🤖 Automation — proactive (cron → Telegram ได้แล้ว)
- [ ] **PR digest เช้า** — "มี PR รอ review, ตัวไหน block release" (ใช้ GitHub MCP)
- [ ] **Standup prep** — สรุปเมื่อวาน + วันนี้จะทำอะไร
- [ ] **EOD → Daily note** อัตโนมัติลง Obsidian (มีโครง EOD cron แล้ว ทำให้ครบ)

### 🧠 Intelligence — ปิด learning loop
- [ ] **Auto-generate SKILL.md** จาก pattern ที่กด 👍 → workflow ซ้ำได้กลายเป็น skill
- [ ] **Feedback → ปรับ persona** — 👎 เยอะเรื่องไหน distill เข้า SOUL.md

### ⚡ Perf — ถ้าอยากเร็วกว่านี้
- [ ] **ลด SDK overhead** — โหลด 92 skills + 4 MCP + system prompt ใหญ่ทุก request (~15s) → lazy-load / ตัด MCP ที่ไม่ใช้ / warm pool

### ❌ ยกเลิก
- ~~Brave Search MCP~~ — ผู้ใช้ยกเลิก
- ~~cloudflared ถาวร~~ — quick tunnel พอแล้ว (มี Telegram เป็นหลัก)

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
