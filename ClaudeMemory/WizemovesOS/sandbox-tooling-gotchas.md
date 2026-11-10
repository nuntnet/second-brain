---
name: sandbox-tooling-gotchas
description: Workarounds for npm/tsx/docker/turbo failures inside the Claude Code sandbox on this repo
metadata:
  node_type: memory
  type: reference
  originSessionId: 202c78fd-87b6-44c7-bd9e-ce11e2a73ba1
  modified: 2026-11-10T14:35:23.797Z
---

- `npm install` fails EPERM (`~/.npm` has root-owned files and is outside the sandbox write list): pass `--cache "$TMPDIR/npm-cache"` and allow `registry.npmjs.org`. Use plain `npm install`, not `-w <pkg>` alone, which left `node_modules/.bin` without turbo/tsc/vite.
- `npx turbo run X --force` gets `--force` eaten by npm ("Missing script: turbo"): run `./node_modules/.bin/turbo run X --force`.
- `npm test` (`tsx --test`) fails EPERM listening on a tsx IPC pipe: use `node --import tsx --test "src/**/*.test.ts"` from the workspace dir.
- Docker socket is blocked by the sandbox, so `docker build` cannot be verified here; tell the user to run it.
- A hook rewrites Bash commands to `rtk ...`; if auto mode complains the input changed, re-issue the exact hook-produced command alone in the next response.
- `rtk read`/`rtk grep` truncate output oddly; prefer the Read tool for config files.
- macOS `mktemp -d` (no template) ignores `$TMPDIR` and is blocked; scripts using it (scripts/deploy/test.sh, validate-k8s.sh) need a PATH shim that calls `/usr/bin/mktemp -d "$TMPDIR/tmp.XXXXXXXXXX"`.
- Embedded Postgres (`npm run verify:migrations`, `test:db`) fails in the sandbox (`shmget` not permitted); these scripts are local-only by design, so they were run with the sandbox disabled for that one command. Needs `npm ci` from main's lockfile first (embedded-postgres, pg).
- Go binaries (glab, helm, tsh) fail TLS verification in the sandbox (macOS keychain, x509 OSStatus -26276). The local Node here is v26 and some web/mobile tests fail on `@wizemoveos/types` named exports; CI uses node:20.
- `git checkout -b x origin/main` through the rtk wrapper once left HEAD on the old branch with main's files staged; verify with `git symbolic-ref HEAD` and fix with `git symbolic-ref HEAD refs/heads/<new>` (worktree untouched).
- Turbo's telemetry host (`telemetry.vercel.com`) is denied; harmless noise.

**Why:** each of these cost a failed attempt on 2026-11-10.
**How to apply:** reach for these workarounds first instead of retrying or using `dangerouslyDisableSandbox`.
