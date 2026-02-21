---
title: Cloudflare & Tailscale – Intern
publish: false
date: 2026-02-20
tags:
  - homelab
  - tailscale
  - cloudflare
  - credentials
---

# Cloudflare & Tailscale – Intern

> ⛔ **Nicht veröffentlichen.** Enthält Tunnel-IDs, Tailnet-Namen und Netzwerkkonfiguration. Öffentliche Version: [[Cloudflare Tunnel & Portainer]].

---

## Tailscale

| Eigenschaft | Wert |
|-------------|------|
| Tailnet | <!-- z.B. tail4a9632.ts.net --> |
| Pi Hostname | <!-- z.B. raspberry-pi-4.tail_____.ts.net --> |
| MagicDNS | aktiv |
| Key Expiry | <!-- wie konfiguriert? --> |

### Tailscale-Geräte

| Gerät | Tailscale-Hostname | OS | Notizen |
|-------|-------------------|-----|---------|
| Raspberry Pi 4 | | Linux | Homeserver |
| MacBook | | macOS | |
| iPhone | | iOS | |
| iPad | | iOS | Dashboard |
| <!-- weitere --> | | | |

### Tailscale ACLs / Zugriff

<!-- Falls du ACLs konfiguriert hast, hier dokumentieren -->

---

## Cloudflare

### Account

| Eigenschaft | Wert |
|-------------|------|
| Account-E-Mail | <!-- hier eintragen --> |
| Domains | saschafiedler.com, <!-- weitere? --> |

### Tunnel

| Eigenschaft | Wert |
|-------------|------|
| Tunnel-Name | <!-- --> |
| Tunnel-ID | <!-- --> |
| Tunnel-Token | In `.env` als `CLOUDFLARED_TUNNEL_TOKEN` |

### Tunnel-Routing (welcher Service über welche Subdomain)

| Subdomain | Ziel-Service | Port |
|-----------|-------------|------|
| <!-- z.B. ha.domain.com --> | Home Assistant | 8123 |
| | | |

### DNS-Records (Cloudflare)

| Record | Typ | Wert | Proxy |
|--------|-----|------|-------|
| `cloud.saschafiedler.com` | CNAME | `hippocamp.uberspace.de` | Proxied |
| `saschafiedler.com` | MX | `hippocamp.uberspace.de` | DNS only |
| <!-- weitere --> | | | |

### Cloudflare-Einstellungen

- SSL-Modus: **Full** (für Nextcloud auf Uberspace)
- WAF: Standard-Regeln aktiv
- Optional: Admin-Bereich per Custom Rule auf eigene IP beschränken

---

## Portainer

| Eigenschaft | Wert |
|-------------|------|
| URL (lokal) | `https://<pi-ip>:9443` |
| URL (Tailscale) | <!-- Tailscale-URL --> |
| Admin-User | <!-- --> |
| Admin-Passwort | <!-- --> |
