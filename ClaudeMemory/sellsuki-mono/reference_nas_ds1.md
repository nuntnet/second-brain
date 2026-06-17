---
name: reference-nas-ds1
description: "How to SSH into the user's Synology NAS (DS1) — Tailscale only, port 2022"
metadata: 
  node_type: memory
  type: reference
  originSessionId: fa60f183-a714-4e6c-bbea-d832eec8343c
---

Synology NAS **DS1** (ch-erawan) is reachable ONLY via Tailscale: `ssh -p 2022 nunt@100.65.67.121` (MagicDNS name `ds1`, tailnet `nuntnet@`). Key auth is set up from this Mac (`~/.ssh/id_ed25519`).

Not reachable from the public internet: `ds1.ch-erawan.com` is Cloudflare-proxied (web only), QuickConnect (`erawan1.sg3.quickconnect.to`) relays only the DSM web UI, and the router (WAN 1.10.230.1, DDNS `cherawan1.synology.me`) has no port forwarding. LAN IP is 192.168.0.10. DSM web UI works at https://erawan1.sg3.quickconnect.to
