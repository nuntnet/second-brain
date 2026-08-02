---
name: reference-caddy-host-networking-gotcha
description: "Caddy container (network_mode:host) stops routing *.sellsuki.local if Docker Desktop's host-networking got enabled after the container was created"
metadata: 
  node_type: memory
  type: reference
  originSessionId: a862be5f-18d0-42e0-a03d-7e69c1c914b9
  modified: 2026-08-02T16:23:24.830Z
---

Symptom: all `*.sellsuki.local` domains (e.g. `provider.sellsuki.local`, `company.sellsuki.local`) fail with `curl: (7) Failed to connect ... Connection refused` on port 443/80, even though `docker logs sellsuki_mono-caddy-1` shows Caddy happily "serving initial configuration" and auto-HTTPS enabled for all the right domains, and `/etc/hosts` correctly maps them to 127.0.0.1.

Root cause: `caddy` service in `docker-compose.yml` uses `network_mode: host`. On Docker Desktop for Mac, host networking only actually binds to the Mac's loopback/interfaces if "Host Networking" was enabled in Docker Desktop settings (`~/Library/Group Containers/group.com.docker/settings-store.json` → `"HostNetworkingEnabled": true`) **before** the container was created. A caddy container created earlier (e.g. weeks prior) keeps running in a stale network namespace even after the setting is turned on later — `docker ps` shows it healthy, logs show it listening, but nothing on the Mac host can reach port 443/80.

**Why**: discovered 2026-08-02 — infra had been up via `make dev` for ~9 min, but both `provider.sellsuki.local` and `company.sellsuki.local` (and in fact every `*.sellsuki.local` domain, verified with `space.sellsuki.local` too) were unreachable. `lsof -nP -i :443 -i :80` on the host showed nothing listening, confirming the container's host network wasn't actually attached to the Mac.

**How to apply**: if any `*.sellsuki.local` domain suddenly can't connect (not 404/502, but connection refused) while caddy's own logs look fine, don't debug Caddyfile routing first — run `docker compose up -d --force-recreate caddy` to force it to rebind under the current Docker Desktop network mode, then retest with `curl -sk -o /dev/null -w "%{http_code}\n" https://<domain>.sellsuki.local/`.
