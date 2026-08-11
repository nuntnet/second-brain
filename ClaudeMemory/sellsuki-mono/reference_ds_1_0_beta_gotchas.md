---
name: ds-1-0-beta-gotchas
description: "@sellsuki-org/sellsuki-components DS 1.0 beta — the semver inversion, the provider-scoped font token, and which tokens are actually published"
metadata:
  node_type: memory
  type: reference
  originSessionId: 72e79abc-11d5-45a2-b332-ae75f50bceb1
  modified: 2026-08-11T12:05:19.591Z
---

Adopting the DS 1.0 beta (`@sellsuki-org/sellsuki-components`) in `ai-chat-admin-frontend` on 2026-08-11 hit four traps worth knowing before any other service upgrades.

**1. The version numbers lie in three different ways.**
- npm orders `0.27.0-beta.1` **below** the older stable `0.27.0`, so `^0.27.0` silently resolves *backwards* off the DS 1.0 line. **Pin exact.**
- The git tag `v0.27.0-beta.1` contains a `package.json` saying `0.27.1-beta.1`. Tag name ≠ package version.
- That `0.27.1-beta.1` is **not published to npm** — the newest published beta is `0.27.0-beta.1`, even though the release note tells people to install `0.26.1-beta.1`.

Despite the lower semver, the beta is a content **superset** of stable: 99 custom elements vs 96 (adds `ssk-clear`, `ssk-dropdown-listbox`, `ssk-search`; removes none). Check this with a diff of `"ssk-*"` strings in the dist bundles before any DS upgrade — it's the cheap regression gate.

**2. `--ssk-font-family-sans` is provider-scoped, not global.** `<ssk-theme-provider>` defines it **on itself**, so it resolves on the provider and everything below, but is **unset on `html`/`body`** (they sit above it). Declaring `font-family: var(--ssk-font-family-sans, system-ui)` on `body` therefore always falls through to the fallback — DS components paint the DS face inside their shadow roots while plain host-page text renders in system-ui. Two typefaces on one screen, and invisible because each half looks fine alone. Declare inherited typography on `ssk-theme-provider` instead. Assume the same trap for any other DS token that reads as unset on `html`.

**3. The `--ssk-type-<role>` tokens aren't shipped yet.** DS 1.0 source adds semantic role tokens (`font: var(--ssk-type-body)` — display/heading/title/subtitle/body-lg/body/body-sm/label/caption, i.e. exactly the model `feature-dod.md` asks for). They exist only in the unpublished `0.27.1-beta.1`; grep the tarball for `ssk-type` before writing code against them.

**4. The DS's own `claude-templates/skills/*.md` were shipped unchanged through the font migration** — byte-identical between 0.27.0 and the beta. So `ssk-tokens.md:371` still says `sans: ["Inter", "DB HeaventRounded", "sans-serif"]`. The runtime token beats the doc; the real value is `"Noto Sans Thai", "Roboto", "sans-serif"`.

Verification note: none of this shows up in a build. Type-check, lint and build all passed while the whole app was rendering in the wrong font — it took reading `getComputedStyle` in a real browser to see it. See [[verify-as-the-user-sees-it]].
