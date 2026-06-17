---
name: reference_secondbrain_vault
description: Obsidian vault at ~/SecondBrain — Claude Code memory is symlinked into it; vault auto-syncs via launchd every 15 min
metadata: 
  node_type: memory
  type: reference
  originSessionId: 2fddc56a-928a-44ed-9f44-644119e31799
---

User's Obsidian vault lives at `/Users/nunt/SecondBrain` (PARA structure, git repo, remote `github.com/nuntnet/second-brain`).

Claude Code's per-project memory is **symlinked** into the vault under `~/SecondBrain/ClaudeMemory/<project>/` (one subfolder per project, kept separate so each project's MEMORY.md doesn't collide). Symlinked projects so far: `sellsuki-mono`, `BOLA`, `WizemovesOS`, `Shipmunk`, `hermes`. e.g.
`~/.claude/projects/-Users-nunt-sellsuki-mono/memory` → `~/SecondBrain/ClaudeMemory/sellsuki-mono/`
So memory files written here land in the vault and are browsable/linkable in Obsidian. No Obsidian app or MCP needed — plain files. To add another project: move its `~/.claude/projects/<dir>/memory` contents into `ClaudeMemory/<name>/`, rmdir, then symlink back.

Vault backup is automated by a launchd agent **`ai.hermes.vault-sync`** (`~/Library/LaunchAgents/ai.hermes.vault-sync.plist`) running `~/.hermes/scripts/sync-vault.sh` every 900s (15 min); logs at `~/.hermes/logs/vault-sync.log`. So a dedicated SessionEnd commit/push hook is redundant.

Memory-discipline `Stop` hook installed at `~/.claude/hooks/memory-save-nudge.sh` (global settings.json): fires once per session at the 3rd turn, fail-open, forces a memory review.

Related: [[project_helio]] (hermes is part of the Helio platform).
