---
name: reference_browser_surfaces_this_workspace
description: "In this workspace the Browser pane times out — use Claude in Chrome, and the user will open a logged-in tab on request"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 369f9f55-f1f1-4ed5-92cb-b4d1e841aa8f
  modified: 2026-08-14T11:00:41.577Z
---

Two browser surfaces exist here and they are NOT interchangeable (2026-08-14):

- **`mcp__Claude_Browser__*` (the in-app Browser pane) — times out.**
  `preview_start` failed after the full 300s on `https://chat.sellsuki.local`,
  and a later `Page.captureScreenshot` timed out at 30s. A PostToolUse hook also
  warns that another chat's dev server owns the folder, so the pane "won't reach
  it". Do not spend a turn on it; five minutes of dead wall-clock.
- **`mcp__claude-in-chrome__*` — works.** It drives the user's real Chrome, so
  their existing Kratos/SSO cookies apply and pages load **already logged in**.

Mechanics worth remembering: `tabs_context_mcp` starts with **no tab group** —
call it with `createIfEmpty: true`, then `navigate` the tab it hands back. The
user's tab is not in your group, so you navigate to the URL yourself; the
session still comes from their profile. `read_network_requests` and
`read_console_messages` only start capturing **when first called**, so an early
page-load failure is invisible — call them, THEN reload, or you will conclude
"no errors" from an empty buffer.

**The user actively supports this.** Asked to check something in a browser, they
opened the URL in their own Chrome and said so ("ผมเปิด … ในแท็ปที่คุณเข้าถึง
แล้วจะได้เห็นบั๊กเต็มๆ"). This is the way past the fact that Claude may not enter
passwords: ask them to open the logged-in page, then inspect and drive it. It
closed out two real bugs that curl-level checks had rated green.

Related: [[feedback_verify_as_the_user_sees_it]]

More quirks (2026-08-14): (1) physical clicks via `computer` are occasionally
swallowed right after a navigation/HMR reload (mousedown/mouseup across a
re-render or focus race) — a "clicked but nothing happened" is NOT app
breakage until a `javascript_tool` `.click()` also fails; JS clicks were 100%
reliable. (2) The page clock can render in a non-local timezone (saw PDT while
the Mac was +07) — likely CDP timezone emulation; don't chase "wrong
timestamps" in app code without checking `new Date()` in the page first.
