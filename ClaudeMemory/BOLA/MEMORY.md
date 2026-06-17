# BOLA Project Memory

## Sellsuki Monorepo Reference
- [Monorepo vs Standalone mapping, ports, known issues](reference_sellsuki_mono.md)

## Database Setup
- **All environments (dev/staging/prod)**: PostgreSQL
- Local dev uses the monorepo's shared Docker PostgreSQL: `postgres://postgres:postgres@localhost:5432/bola?sslmode=disable`
- SQLite is no longer used — all 30 migration files updated from `DATETIME` → `TIMESTAMP`
- `DATABASE_URL` is the correct env var (was `DATABASE_URI` — fixed in `.env`)

## Migration Rules
- ❌ DON'T: `gen_random_uuid()`, `NOW()`, `DATETIME`, `JSONB`, `BOOLEAN`, `TIMESTAMPTZ`
- ✅ DO: `CURRENT_TIMESTAMP`, `TEXT` for JSON, `TIMESTAMP` for timestamps
- ✅ Generate UUIDs in Go code, not SQL
- Test migrations: `DATABASE_URL="postgres://postgres:postgres@localhost:5432/bola?sslmode=disable" go run ./cmd/migration/... --ci`

## Key Info
- Stack: Go backend, React frontend
- Project pattern: Clean Architecture with interfaces
- Docs: Refer to `/docs` first before code

## Standing Workflow Rule (User Preference)
After every successful dev cycle (code complete + tests pass + merge):
- **Auto-update docs without being asked** — this is mandatory, part of Definition of Done
- Update order: spec.md → endpoints.md → schema.md → TODO.md (✅) → ROADMAP.md
- Commit docs together with code, never leave them stale
- User only needs to say `"Implement [feature]"` — Claude handles everything else automatically

## Git Repositories (Updated)
- **Backend**: `/Users/nunt/BOLA/backend/`
  - Remote: `https://gitlab.sellsuki.com/sellsuki/bola/back-office-of-line-api-backend` (branch: `main`)
  - Status: ✅ All commits pushed successfully
- **Frontend**: `/Users/nunt/BOLA/frontend/`
  - Remote: `https://gitlab.sellsuki.com/sellsuki/bola/back-office-of-line-api-frontend` (branch: `master`)
  - Status: ✅ All commits pushed successfully
- **Docs**: `/Users/nunt/BOLA/docs/` (not in git - reference only)

## Environment Configuration
### Webhook Base URL (WEBHOOK_BASE_URL)
- **Purpose**: Base URL for generating webhook endpoints (e.g., used in auto push messages)
- **Default**: `http://localhost:8080` (from backend/cmd/bola_server/main.go)
- **For ngrok tunneling**: Set `WEBHOOK_BASE_URL=https://your-ngrok-url.ngrok.io`
- **Example**: `WEBHOOK_BASE_URL="https://abc123.ngrok.io" go run ./cmd/bola_server/main.go`
- The frontend receives this value from the backend API response

## Frontend Redirect Behavior
- After creating resource (auto push message, webhook, etc.), should redirect to detail page
- Use `window.location.href = "/resource-type/{id}"` for navigation
- Currently: CreateAutoPushDialog closes and adds to list, but doesn't redirect to detail page

## Media Library (Completed 2026-03-10)
- Migration 0035: extends media table with alt_text, action_url, width, height, tags, upload_status, thumbnail_s3_key, usage_count, updated_at, deleted_at
- StorageService interface + MinIO/S3 impl + NoopStorage fallback
- StorageService injected via `SetStorageService()` setter (same as SetLLMService pattern)
- Soft delete with 7-day trash, ErrInUse guard, usage tracking
- Frontend: Canvas API client-side resize + UploadMediaDialog + EditMediaDialog + Tabs
- Use case coverage: 71.6% — Next migration: 0036

## PM Agent Ecosystem (Added 2026-03-11)

### Agent Files
- `.claude/agents/product-manager.md` — Ploy persona, ICE scoring + extended v2: user manual, video tutorial, release notes (SRE gate), roadmap balance (60/40 rule)
- `.claude/agents/picky-customer.md` — Wanchai persona, 37-OA agency owner
- CLAUDE.md: `🎨 UX/UI Agent` + `🛍️ Picky Customer & Product Manager Agents` sections

### Agent Report Storage
- `docs/agent-reports/uxui-latest.md` — overwritten each `/uxui` run
- `docs/agent-reports/picky-latest.md` — overwritten each `/picky-customer review` run
- PM Agent reads these for `/pm roadmap-balance`

### Landing Page
- Static site: `/Users/nunt/BOLA/landing/` (index.html + css/styles.css + js/main.js)
- Tailwind CDN + Noto Sans Thai, LINE green #06C755, Thai SMB/enterprise content
- What's New section updated by `/pm release-note`

### In-App Features Added
- `FeedbackWidget.tsx` — fixed floating button + Dialog, localStorage key `bola_feedback_entries`
- `UserManualPage.tsx` — 6 Thai guides, search, helpful votes, route `/user-manual`
- Sidebar: "User Manual" link with BookMarked icon

### Docs Directories
- `docs/user-manual/` — 7 files (README + getting-started, line-oa, broadcasts, segments, ai-chatbot, auto-reply)
- `docs/release-notes/` — README (SRE [PENDING APPROVAL]→[APPROVED] process) + v1.0 release
- `docs/video-tutorials/` — README (video production guide)

## Test Coverage Progress

### Session 3 Summary: Approaching 70% Target (Current Session)

**Starting Coverage (Session 3)**: 65.0% (from previous session)
**Current Coverage**: 69.6%
**Session 3 Improvement**: +4.6% 🎯
**Overall Project Improvement**: 55.2% → 69.6% (+14.4% total!)
**Remaining to 70% Target**: 0.4% (95% of the way there!)

**Tests Added in Session 3** (20+ new test cases):
1. lon_consent_test.go: GetLONPublicOAInfo (3 tests)
2. use_case_test.go: SetSyncDeliveryThreshold, SetBroadcastSyncDeliveryThreshold, SetLLMService (7 tests)
3. rich_menu_assignment_test.go: UpdateRichMenuAssignment (4 tests)
4. workspace_test.go: UpdateWorkspace errors + multiple fields (4 tests)
5. auto_reply_test.go: UpdateAutoReply error paths (2 tests)
6. segment_test.go: UpdateSegment error (1 test)

**Key Achievement**: All functions now have at least some test coverage (0% → minimized)

---

### Session 2 Summary: Bridge to 70% Target

**Starting Coverage (Session 2)**: 62.5% (from previous session)
**Previous Session Coverage**: 65.8%
**Session 2 Improvement**: +3.3% (continuing momentum)
**Overall Project Improvement at Session 2 End**: 55.2% → 65.8% (+10.6% total)

**Tests Added in Session 2**:

1. **Auto Reply Tests** (5 new tests):
   - GetAutoReply: success and not found cases
   - ListAutoReplies: non-empty, empty, and error cases
   - Coverage: +0.3% (65.0% → 65.3%)

2. **Outbound Event Tests** (8 tests in new file):
   - UpdateOutboundWebhook: success, not found, repository error
   - ListOutboundDeliveryLogs: pagination, empty list, error handling
   - Coverage: +0.3% (65.3% → 65.6%)

3. **Segment Tests** (1 additional test):
   - DeleteSegment: not found error case (added to existing test file)
   - Coverage: +0.2% (65.6% → 65.8%)

**Total Tests Added Session 2**: 14 new test cases
**Files Created/Modified**:
- outbound_event_test.go (238 lines, 8 tests) — NEW
- auto_reply_test.go (added 5 tests)
- segment_test.go (added 1 error case test)

**Test Coverage Statistics**:
- Total test functions in project: 740+
- Use case test coverage: 65.8%
- Coverage gap to 70% target: 4.2%

**Key Observations**:
- Most main use case methods have comprehensive tests
- Remaining gap likely from:
  - Private/internal helper functions (~1-2%)
  - Edge cases in existing tested functions (~2-3%)
  - Less critical utility functions
- Decorator/wrapper functions already well tested (auto_reply, segment, etc.)

**Remaining Gap to 70%**: 4.2% (~60-70 additional test cases estimated)

**High-Value Targets for Final Push**:
1. Rich menu scheduler functions (if any)
2. Broadcast scheduler edge cases
3. Auto push message edge cases
4. Helper function edge cases across modules
