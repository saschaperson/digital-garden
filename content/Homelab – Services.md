---
title: Homelab – Services
publish: true
date: 2026-03-15
tags:
  - homelab
  - actual-budget
  - isponsorblocktv
---

# Homelab – Services

> CT 104 auf dem Proxmox-Host. Leichtgewichtige Services die keinen eigenen Container rechtfertigen. Zurück zum [[Homelab]]-Überblick.

---

## Container-Konfiguration

| Eigenschaft | Wert |
|-------------|------|
| CT ID | 104 |
| Hostname | `services` |
| IP | 192.168.178.13 |
| RAM | 512 MB |
| Disk | 4 GB (ZFS Subvolume) |
| Cores | 1 |
| Mount | `/mnt/storage` → `/mnt/media` (read-only) |

### Compose

- Pfad: `/opt/services/docker-compose.yml`
- Kein `.env`-File — keine Secrets nötig

---

## Actual Budget

Self-hosted Budgeting-Tool. Offline-first, schnell, mit optionalem Sync zwischen Geräten.

| Eigenschaft | Wert |
|-------------|------|
| Image | `actualbudget/actual-server:latest` |
| Port | 5006 |
| Daten | `/opt/services/actual/` (server-files, user-files) |
| Extern | Nicht exponiert — nur LAN und Tailscale |

Actual ist bewusst nicht im Cloudflare Tunnel. Zugriff von unterwegs ausschließlich über Tailscale (`http://192.168.178.13:5006`).

---

## iSponsorBlockTV

Überspringt Sponsor-Segmente in YouTube-Videos auf dem Apple TV. Nutzt die SponsorBlock-Community-Datenbank.

| Eigenschaft | Wert |
|-------------|------|
| Image | `ghcr.io/dmunozv04/isponsorblocktv:latest` |
| Netzwerk | `network_mode: host` (für Apple TV Discovery via mDNS) |
| Config | `/opt/services/isponsorblocktv/` |
| Ports | Keine — reiner Client |

---

## Troubleshooting

- **Actual nicht erreichbar von unterwegs:** Tailscale aktiv? `http://192.168.178.13:5006` über Tailscale-IP testen
- **iSponsorBlockTV überspringt nicht:** `docker logs isponsorblocktv --tail 20` — Apple TV muss im selben Subnet sein und gekoppelt sein
- **Apple TV nicht gefunden nach Neustart:** Container neu starten, ggf. neu koppeln
