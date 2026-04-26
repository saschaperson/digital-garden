---
title: Homelab – Services
publish: true
date: 2026-04-26
tags:
  - homelab
  - services
  - docker
---

# Homelab – Services

> CT 104 — Leichtgewichtige Services ohne eigene Isolation. Zurück zum [[Homelab]]-Überblick.

---

## Container

| Eigenschaft | Wert |
|-------------|------|
| CT ID | 104 |
| Hostname | `services` |
| Cores | 2 |
| RAM | 2 GB |
| Swap | 512 MB |
| Disk | 12 GB (local-zfs) |
| Bind-Mount | `/mnt/storage` → `/mnt/media` (ro) |
| Start Order | 4 |

Services die keine eigene Isolation brauchen und wenig Ressourcen verbrauchen. Jeder Service hat ein eigenes Compose-File unter `/opt/<service>/`. Ausnahme: Actual, iSponsorBlockTV, Bambuddy und Portainer Agent teilen sich `/opt/services/docker-compose.yml` (historisch gewachsen, wird nicht refactored).

---

## Services

| Service | Port | Funktion |
|---------|------|----------|
| Actual Budget | 5006 | Budgetverwaltung |
| iSponsorBlockTV | host network | SponsorBlock für Apple TV |
| Bambuddy | 8000 | Bambu Lab A1 Drucker-Monitoring |
| Speedtest Tracker | 8765 | WAN-Speedtests alle 2 Stunden |
| Homarr | 7575 | Dashboard (Admin-Board + Freunde-Board) |
| BetterBahn | 3000 | DB Split-Ticketing |
| Homebox | 3100 | Haushaltsinventar mit QR-Codes |
| Portainer Agent | 9001 | Remote-Management für Portainer auf CT 101 |

---

## Actual Budget

Benötigt SharedArrayBuffer, das nur über HTTPS funktioniert. Gelöst über Tailscale Serve auf dem Proxmox-Host — stellt den Service unter einer Tailscale-HTTPS-URL mit gültigem Zertifikat bereit. Nicht über Cloudflare Tunnel exponiert.

```bash
tailscale serve --bg --https=5006 http://<ct-ip>:5006
```

Die Serve-Konfiguration überlebt Reboots automatisch.

---

## Homarr

Multi-User-Boards: Admin-Board mit allen Services, Freunde-Board mit Jellyfin, Seerr, BetterBahn. Proxmox-Integration über dedizierter API-Token mit PVEAuditor-Rolle.

Extern über Cloudflare Tunnel, eigener Login (kein Cloudflare Access davor).

---

## BetterBahn

Kein eigener Login. Deshalb Cloudflare Access mit E-Mail-OTP davor — sonst wäre der Endpunkt offen für Bot-Traffic der die DB-API überlasten kann.

---

## Compose-Files

```yaml
# /opt/services/docker-compose.yml
services:
  actual:
    image: actualbudget/actual-server:latest
    container_name: actual
    restart: unless-stopped
    ports:
      - "5006:5006"
    volumes:
      - ./actual:/data

  isponsorblocktv:
    image: ghcr.io/dmunozv04/isponsorblocktv:latest
    container_name: isponsorblocktv
    restart: unless-stopped
    network_mode: host
    volumes:
      - ./isponsorblocktv:/app/data

  bambuddy:
    image: ghcr.io/maziggy/bambuddy:latest
    container_name: bambuddy
    user: "1000:1000"
    cap_add:
      - NET_BIND_SERVICE
    network_mode: host
    environment:
      - TZ=Europe/Berlin
      - PORT=8000
    volumes:
      - /opt/services/bambuddy/data:/app/data
      - /opt/services/bambuddy/logs:/app/logs
    restart: unless-stopped

  portainer-agent:
    image: portainer/agent:lts
    container_name: portainer-agent
    restart: unless-stopped
    ports:
      - "9001:9001"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
```

```yaml
# /opt/speedtest-tracker/docker-compose.yml
services:
  speedtest-tracker:
    image: lscr.io/linuxserver/speedtest-tracker:latest
    container_name: speedtest-tracker
    restart: unless-stopped
    ports:
      - "8765:80"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
      - APP_KEY=${SPEEDTEST_APP_KEY}
      - DB_CONNECTION=sqlite
      - SPEEDTEST_SCHEDULE=0 */2 * * *
      - DISPLAY_TIMEZONE=Europe/Berlin
    volumes:
      - ./config:/config
```

```yaml
# /opt/homarr/docker-compose.yml
services:
  homarr:
    image: ghcr.io/homarr-labs/homarr:latest
    container_name: homarr
    restart: unless-stopped
    ports:
      - "7575:7575"
    environment:
      - TZ=Europe/Berlin
      - SECRET_ENCRYPTION_KEY=${HOMARR_SECRET}
    volumes:
      - ./appdata:/appdata
```

```yaml
# /opt/betterbahn/docker-compose.yml
services:
  betterbahn:
    image: ghcr.io/betterbahn/betterbahn:latest
    container_name: betterbahn
    restart: unless-stopped
    environment:
      TZ: Europe/Berlin
      NODE_ENV: production
      NEXT_TELEMETRY_DISABLED: "1"
    ports:
      - "3000:3000"
```

```yaml
# /opt/homebox/docker-compose.yml
services:
  homebox:
    image: ghcr.io/sysadminsmedia/homebox:latest
    container_name: homebox
    restart: unless-stopped
    ports:
      - "3100:7745"
    environment:
      - TZ=Europe/Berlin
      - HBOX_LOG_LEVEL=info
      - HBOX_LOG_FORMAT=text
      - HBOX_WEB_MAX_UPLOAD_SIZE=10
      - HBOX_OPTIONS_ALLOW_ANALYTICS=false
    volumes:
      - ./data:/data
```

Secrets in `.env`-Dateien im jeweiligen Verzeichnis.
