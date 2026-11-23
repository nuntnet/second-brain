---
name: reference-console-new-request-header-needs-chat-core-cors
description: "A new request header from the AI chat console (e.g. kit `traceparent`, AI-402) is dropped by the browser unless chat-core's hard-coded CORS AllowHeaders lists it — staging console was down 2026-11-23 ~12:00–13:49"
metadata:
  node_type: memory
  type: reference
  originSessionId: c0bbaaf9-c856-4741-a704-2b0a7e55e366
  modified: 2026-11-23T08:34:23.213Z
---

chat-core CORS `AllowHeaders` is hard-coded in `src/interface/fiber_server/fiber_server.go` (no env). When console !184 (AI-402 kit fetcher) started sending `traceparent` on every call, the preflight still answered 204 but the browser dropped the real request: `GET /v1/me` → "Failed to fetch" → console AuthGuard shows "ตอนนี้เปิดระบบไม่ได้" (SESSION_LOOKUP_FAILED, not the login redirect). Console tests use mocks, so nothing caught it; the AI-402 dev check ("every request shows traceparent") was Not done.

Fixed by chat-core !268 → v1.3.3 (`traceparent,tracestate` added, test `TestAI402_CORSPreflight_AllowsTraceContextHeaders`). Console v1.2.1 prod deploy was cancelled until chat-core v1.3.3 was on prod.

**How to apply:** any console change that adds a request header ships the chat-core CORS change FIRST (producer before consumer). Check with `curl -X OPTIONS -H "Origin: <console>" -H "Access-Control-Request-Headers: <hdr>" <chat-core>/v1/me` and read `access-control-allow-headers`. Diagnose "can't open" screens by reading the ErrorState's `error` prop via the React fiber — it carries the real message. Related: [[feedback-verify-as-the-user-sees-it]].
