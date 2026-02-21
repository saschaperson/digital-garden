---
title: Docker Compose – Homeserver
publish: true
date: 2026-02-20
tags:
  - homelab
  - docker
  - docker-compose
---
# Docker Compose – Homeserver

> Zurück zum [[Homeserver]]-Überblick.

Die `docker-compose.yml` für alle öffentlich dokumentierten Services auf dem Raspberry Pi 4. Alle sensiblen Werte kommen aus einer `.env`-Datei (siehe [[Docker .env – Homeserver]], nicht öffentlich). Weitere Services sind im [[Servarr]]-Zettel dokumentiert (ebenfalls nicht öffentlich).

---

## Compose-Datei

```yaml
services:
  pihole:
    image: jacklul/pihole:latest
    container_name: pihole
    hostname: pihole
    network_mode: host
    environment:
      TZ: ${TZ}
      FTLCONF_webserver_api_password: "${PIHOLE_FTL_API_PASSWORD}"
      BLOCKLISTS_URL: "https://v.firebog.net/hosts/lists.php?type=tick"
      REGEX_BLACKLIST_URL: "https://raw.githubusercontent.com/mmotti/pihole-regex/master/regex.list"
    volumes:
      - ./pihole/etc-pihole:/etc/pihole
    restart: unless-stopped

  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    network_mode: host
    environment:
      TZ: ${TZ}
    volumes:
      - ./homeassistant:/config
    devices:
      - /dev/ttyAMA0:/dev/ttyAMA0   # RaspBee II Zigbee Stick
    restart: unless-stopped

  isponsorblocktv:
    image: ghcr.io/dmunozv04/isponsorblocktv:latest
    container_name: isponsorblocktv
    environment:
      TZ: ${TZ}
    volumes:
      - ./isponsorblocktv:/app/data
    restart: unless-stopped

  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    environment:
      TUNNEL_TOKEN: "${CLOUDFLARED_TUNNEL_TOKEN}"
    command: tunnel --no-autoupdate run
    restart: unless-stopped

  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    network_mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer_data:/data
    restart: unless-stopped

  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      TZ: ${TZ}
      DOMAIN: "${VW_DOMAIN}"
      SIGNUPS_ALLOWED: "${VW_SIGNUPS_ALLOWED}"
      SIGNUPS_VERIFY: "false"
      ADMIN_TOKEN: "${VW_ADMIN_TOKEN}"
    volumes:
      - ./vaultwarden/data:/data
    ports:
      - "8081:80"

  actual:
    image: actualbudget/actual-server:latest
    container_name: actual
    restart: unless-stopped
    environment:
      TZ: ${TZ}
    volumes:
      - ./actual:/data
    ports:
      - "5006:5006"

  bambuddy:
    image: ghcr.io/maziggy/bambuddy:latest
    container_name: bambuddy
    network_mode: host
    user: "${PUID}:${PGID}"
    environment:
      TZ: ${TZ}
      PORT: "8000"
    volumes:
      - bambuddy_data:/app/data
      - bambuddy_logs:/app/logs
    restart: unless-stopped

  # Weitere Services (Jellyfin, Servarr-Stack) sind im
  # privaten Servarr-Zettel dokumentiert.

volumes:
  bambuddy_data:
  bambuddy_logs:
```

---

## .env-Vorlage

Zum Nachbauen – echte Werte nicht im öffentlichen Zettel:

```bash
# Allgemein
TZ=Europe/Berlin
PUID=1000
PGID=1000

# Pi-hole
PIHOLE_FTL_API_PASSWORD=changeme

# Cloudflare Tunnel
CLOUDFLARED_TUNNEL_TOKEN=your-tunnel-token

# Vaultwarden
VW_DOMAIN=https://your-tailscale-hostname
VW_SIGNUPS_ALLOWED=false
VW_ADMIN_TOKEN=changeme
```

---

## Patterns & Entscheidungen

### `network_mode: host` vs. Port-Mapping

| Service | Modus | Grund |
|---------|-------|-------|
| Pi-hole | host | DNS auf Port 53, DHCP-Broadcast |
| Home Assistant | host | mDNS, UPnP, Geräte-Discovery |
| Portainer | host | Direkter Zugriff ohne Port-Konfiguration |
| BamBuddy | host | Printer Discovery, Kamera-Streaming |
| Vaultwarden | ports | Kein Discovery nötig, ein Port reicht |
| Actual | ports | Einfache Web-App, ein Port reicht |
| Cloudflared | bridge | Ausgehende Verbindung, kein Port nötig |

---

## Verwaltung

```bash
# Alle Services starten
cd /home/sascha/docker/homeserver
docker compose up -d

# Einzelnen Service neustarten
docker compose restart homeassistant

# Logs eines Services
docker logs -n 50 -f homeassistant

# Alle Images updaten
docker compose pull && docker compose up -d
```
