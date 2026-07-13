---
name: reference-nas-ds1
description: "How to SSH into the user's Synology NAS (DS1) — Tailscale only, port 2022; how to recover if the Tailscale node drops"
metadata: 
  node_type: memory
  type: reference
  originSessionId: fa60f183-a714-4e6c-bbea-d832eec8343c
---

Synology NAS **DS1** (ch-erawan) is reachable ONLY via Tailscale: `ssh -p 2022 nunt@100.65.67.121` (MagicDNS name `ds1`, tailnet `nuntnet@`). Key auth is set up from this Mac (`~/.ssh/id_ed25519`).

Not reachable from the public internet for SSH: `ds1.ch-erawan.com` is Cloudflare-proxied (web only), QuickConnect (`erawan1.sg3.quickconnect.to`) relays only the DSM web UI, and the router (WAN 1.10.230.1, DDNS `cherawan1.synology.me`) has no port forwarding to DSM (port 443 there only serves a default Web Station placeholder page). LAN IP is 192.168.0.10. DSM web UI works at both https://erawan1.sg3.quickconnect.to and **https://ds1.ch-erawan.com** (confirmed 2026-07-14 — renders the "Sign in to DSM" page branded "Ch.Erawan Group").

**If DS1 disappears entirely from `tailscale status`** (not even listed as offline — happened 2026-07-14, likely a Tailscale node key/session expiry): the device itself is still up and DSM is reachable via the URLs above. Fix: log into DSM at `https://ds1.ch-erawan.com` yourself (do not let Claude enter the DSM password), open **Package Center → Tailscale** (or Control Panel, depending how it's installed), and re-authenticate/reconnect. It reappears in this Mac's `tailscale status` within ~30s, though `tailscale ping`/SSH may briefly say "unknown peer" until the WireGuard handshake completes (~10-20s more) — this is normal, not a fault.

The NAS also runs Docker (`/usr/local/bin/docker`), but the `nunt` SSH user does NOT have permission on `/var/run/docker.sock` (permission denied) despite being in the `administrators` group — `docker ps`/`exec` needs a different privilege path (sudo password, or a docker-group grant) not yet set up. See [[reference_cats_ats_system]] for what's actually deployed there.
