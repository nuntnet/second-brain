---
name: reference-machine-vpn-clients-wireguard-is-company-vpn
description: VPN บนเครื่อง nunt — WireGuard คือ VPN บริษัท (sellsuki-production / sellsuki-staging-th / kruengkao) split-tunnel 10.21/16 + DNS 1.1.1.1,8.8.8.8 + IPv6; Tailscale หยุดอยู่และเวอร์ชันไม่ตรง; sandbox อ่าน ps/route/networksetup ไม่ได้
metadata:
  type: reference
---

ตรวจเมื่อ 2026-10-15 (ถามว่า "ต่อเน็ตอย่างไร"):

- **WireGuard.app คือ VPN บริษัท** มี 3 โปรไฟล์ (อ่านจาก `/Library/Preferences/com.apple.networkextension.plist` ด้วย `plutil -p` — config จริงอยู่ใน Keychain อ่านไม่ได้):
  `sellsuki-production` → vpn.production.sellsuki.com:51820 · `sellsuki-staging-th` → vpn.staging-th.sellsuki.com:51820 · `Nunt-Macbookair` → vpn.svc.kruengkao.com:51820
- ตอนต่อ: utun ได้ IP 100.65.x.x (CGNAT) MTU 1280, **split tunnel**: route เข้าอุโมงค์เฉพาะ 10.21.0.0/16, 1.1.1.1, 8.8.8.8 และ IPv6 default — IPv4 ทั่วไปยังออก Wi‑Fi ตรง → ถ้าอุโมงค์สะดุด DNS ล้มทั้งที่ Wi‑Fi ปกติ
- Tailscale ติดตั้งแต่ `Tailscale is stopped` และ CLI 1.104 ≠ daemon 1.98 (ใช้กับ NAS เท่านั้น ดู [[reference-nas-ds1]], [[reference-outline-mcp-vpn-blocker]])
- OpenVPN Connect + Tunnelblick + Turbo VPN ติดตั้งอยู่แต่ไม่ได้ใช้ (OpenVPN log สุดท้าย 2025-03)
- **sandbox บล็อก** `ps`, `route get`, `networksetup`, `ipconfig getifaddr`, Tailscale CLI (Failed to load preferences) — ใช้ `netstat -rn`, `ifconfig`, `plutil -p …NetworkInterfaces.plist` แทน
- macOS 26: `networksetup -getairportnetwork en0` ตอบ "not associated" ทั้งที่ Wi‑Fi ต่ออยู่ เพราะ Terminal ไม่มีสิทธิ์ Location Services — ไม่ใช่สถานะจริง

**How to apply:** ถาม "VPN ขึ้นไหม" → `netstat -rn -f inet | grep utun` หา route 10.21/16; ถ้าต้องเข้า internal host 10.x ให้ผู้ใช้เปิด WireGuard โปรไฟล์ sellsuki ไม่ใช่ Tailscale
