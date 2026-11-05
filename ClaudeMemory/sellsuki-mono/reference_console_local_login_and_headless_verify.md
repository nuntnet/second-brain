---
name: reference-console-local-login-and-headless-verify
description: AI chat console on local signs in only via https://chat.sellsuki.local (Kratos rejects return_to localhost:5191); agent verifies with headless Playwright + a session cookie from a localhost:4433 Kratos login
metadata:
  node_type: memory
  type: reference
  originSessionId: 70429684-da98-4359-8018-c1ff7471777a
  modified: 2026-11-05T14:49:22.115Z
---

- `http://localhost:5191` cannot sign in: Kratos answers "Requested return_to URL http://localhost:5191/ is not allowed". Only `https://chat.sellsuki.local` (Caddy → 5191) works for a human.
- The built-in browser pane refuses `chat.sellsuki.local` ("denied or failed") until the user allows it; Claude in Chrome may be disconnected.
- Agent-side check that works (2026-11-05): log in against `http://localhost:4433` self-service browser flow with the `kratos` identity from `.claude/verify/.local/credentials.json` (JSON POST, carry the csrf cookie), take the `sellsuki_session` Set-Cookie, then `ctx.addCookies({domain: ".sellsuki.local", secure, httpOnly})` in the console's own `@playwright/test` chromium (run the script from `apps/admin` so the import resolves). chat-core APIs accept the same value as a `Cookie:` header.
- Caddy (`network_mode: host`) can silently stop binding 443 → see [[reference-caddy-host-networking-gotcha]]; `docker compose up -d --force-recreate caddy`.
- The ai-mvp stack runs under its own overmind socket `.overmind-ai-mvp.sock` (tmux `-L overmind-sellsuki-mono-<id>`), separate from `.overmind.sock`.
- Demo data: `scripts/seed-ai-chat-demo.sh` (workspace "บ้านใบไม้ สตูดิโอ (เดโม)" in company 85fa00dd…, only the verify identity has access).
