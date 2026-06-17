# WizemovesOS Memory

## Project Identity
- **Name**: WizemovesOS (Sales + Project Management CRM for Sellsuki/Wizemoves digital agency)
- **GitLab**: `git@gitlab.sellsuki.com:sellsuki/wizemoveos.git`
- **Working dir**: `/Users/nunt/WizemovesOS`

## Architecture (Phase 0 complete)
- **Monorepo**: Turborepo (npm workspaces)
- **apps/web**: React 19 + Vite + TanStack Router + TanStack Query v5 + Zustand + RHF + Zod + Sonner
- **apps/bola**: LINE bot webhook (Phase 7)
- **packages/types**: Shared TypeScript types (all entities)
- **packages/config**: Shared ESLint + TSConfig
- **supabase/**: Migrations + Edge Functions + seed
- **Design system**: `@sellsuki-org/sellsuki-components` (public npm, installed)
- **Backend**: Supabase (Postgres + Auth + Realtime + Edge Functions + Storage)
- **Auth**: Google OAuth restricted to @sellsuki.com domain
- **AI**: Claude API (Haiku for embedding, Sonnet for drafting)
- **Vector DB**: pgvector (Supabase extension — no extra service)

## Database Tables (migration 0001)
users, app_settings, contacts, customer_profiles, leads, deals, projects,
project_tasks, todos, proposals, tech_stack_templates,
proposal_knowledge_items (with vector(1536) embedding),
activities, notifications (Realtime enabled), portfolio_items

## Code Patterns (MUST follow)
- **Server state**: TanStack Query ONLY — no useEffect fetching
- **Forms**: React Hook Form + Zod — no useState for inputs
- **Types**: strict TypeScript, no `any` — use `unknown`
- **Feature structure**: `features/{name}/{api,components,hooks,schemas}/index.ts`
- **API pattern**: `.keys.ts` → `.queries.ts` → `.mutations.ts` per feature
- **No prop drilling**: past 2 levels → use Zustand or query hooks
- **Commits**: Conventional commits (feat:, fix:, chore:, refactor:)

## Key Files
- `apps/web/src/lib/supabase.ts` — Supabase client
- `apps/web/src/lib/query-client.ts` — TanStack Query config
- `apps/web/src/lib/database.types.ts` — DB types (auto-gen from Supabase)
- `packages/types/src/index.ts` — ALL shared types
- `supabase/migrations/0001_initial_schema.sql` — complete DB schema
- `supabase/seed.sql` — default settings + tech stack templates
- `apps/web/src/shared/hooks/useTableProcessing.ts` — shared filter/sort
- `apps/web/src/features/leads/api/` — example of full API pattern

## TypeScript Gotchas (resolved)
- `apps/web/tsconfig.json` extends `../../packages/config/tsconfig/react.json` (relative path — bare `@wizemoveos/config/...` not resolved by tsc)
- `exactOptionalPropertyTypes: true` removed from base.json — too strict with row mappers
- `Relationships: []` required on ALL tables in database.types.ts (Supabase v2.99+ GenericTable)
- Json casts require `as unknown as Json` (two-step) — `StageHistoryEntry[]`, `Record<string, unknown>` etc.
- TanStack Router: `_authenticated` is a PATHLESS layout — nested routes resolve to `/dashboard`, `/deals` NOT `/_authenticated/dashboard`
- Insert types: ALL non-omitted nullable columns must be explicitly provided (e.g., `converted_from_lead_id: null`, `position: null`)

## Implementation Phases (see plan)
- ✅ Phase 0: Foundation (done)
- ✅ Phase 1: Core CRM — fully wired to local Supabase; auth + RLS working
- ✅ Phase 2: Projects + Tasks + Todos — typecheck ✅, build ✅
- ✅ Phase 3: Proposals + AI — typecheck ✅
- ✅ Phase 4: Reports + Dashboard — typecheck ✅
- ✅ Phase 5: Notifications (wired)
- ✅ Phase 6: Portfolio + Knowledge Base + Tech Stack (wired)
- ✅ Phase 7.5: Smart Price Proposal Engine (STORY-701–706: margin panel, AI suggestions, slider, approval workflow, competitor benchmark, file upload)
- ✅ Phase 9: Security Hardening + Data Integrity (RLS, Zod validation, rate limiting, atomic conversion, pagination, indexes)
- ⬜ Phase 7: BOLA LINE Agent
- ⬜ Phase 8: Mobile (React Native + Expo)
- ⬜ Phase 9.5: UX Polish + Architecture Refactor
- ⬜ Phase 10: AI Meeting Intelligence (brief parser, meeting transcript import, meeting prep questions) — spec drafted 2026-03-15

## User Preferences
- All responsive (test 375px / 768px / 1280px)
- Use @sellsuki-org/sellsuki-components FIRST, Tailwind only for layout/spacing
- No hardcoded colors — use design tokens
- Intuitive UX: empty states, confirmations, progress indicators
- Thai Baht (THB) as default currency
- Legacy code in `apps/web-legacy/` — reference only, not deployed

## Local Supabase Dev Setup (DONE)
- Run: `GOOGLE_CLIENT_ID=placeholder GOOGLE_CLIENT_SECRET=placeholder supabase start`
- Reset DB + seed: `supabase db reset --local`
- Studio: http://127.0.0.1:54323 | API: http://127.0.0.1:54321
- `apps/web/.env.local` — has local URL + anon key + `VITE_DEV_BYPASS_AUTH=true`
- Dev user: `dev@sellsuki.com` / `devpassword123` (PM role) — id: `a3bead0d-200c-4cf4-957f-dd17a66654a1`
- `useAuth.ts` auto-signs-in with dev credentials when `VITE_DEV_BYPASS_AUTH=true` so RLS passes
- Worktree dev server: port 3002 (vite.config.ts `strictPort: true`)
- For production: create Supabase cloud project, copy URL+anon key, enable Google OAuth, set ANTHROPIC_API_KEY in edge function secrets

## Product Decisions
- [Field Configuration (Admin-Only)](./product_field_configuration.md) — custom fields configured via migrations, not Settings UI
- [Proposals belong to Deals](./product_proposal_on_deal.md) — Proposals are Deal-centric (pre-sale), not Project-centric. Project created after acceptance.
