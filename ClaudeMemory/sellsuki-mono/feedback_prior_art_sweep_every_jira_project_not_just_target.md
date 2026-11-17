---
name: prior-art-sweep-every-jira-project
description: "Before opening a card, search PAT, AI and OC Jira projects plus read the content of adjacent hits; titles alone mislead"
metadata:
  node_type: memory
  type: feedback
  originSessionId: a0c4bf2e-bdf1-481b-bcff-5c37129e1d40
  modified: 2026-11-17T10:32:01.336Z
---

nunt asked "do those 3 related cards actually concern us?" right after I opened PAT-2872/2873. I had searched only PAT and listed neighbours from titles; reading them showed an existing reconciler (AI-252) and a shared-preset plan (AI-248) living in the AI project that change the cards' scope.

**Why:** this workspace's work is split across PAT, AI and OC boards; the same domain (RBAC, AI chat) has cards in several. A PAT-only search reads as "no duplicate" and is wrong.

**How to apply:** for `shipping.md` rule 11 prior art, run the Jira search per project (PAT, AI, OC, BOLA) and open the content of any adjacent hit before writing a card. Say which projects and terms were searched. Never cite a card as related from its title alone.
