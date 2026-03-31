# Homelab – Services

> CT 104 — Der Sammel-Container für leichtgewichtige Services. Zurück zum [[Homelab]]-Überblick.

---

## Container

| Eigenschaft | Wert |
| --- | --- |
| CT ID | 104 |
| Hostname | `services` |
| IP | 192.168.178.13 |
| Cores | 2 |
| RAM | 2 GB |
| Swap | 512 MB |
| Disk | 12 GB (local-zfs) |
| Bind-Mount | `/mnt/storage` → `/mnt/media` (ro) |
| Start Order | 4 |

Hier landen Tools, die keine eigene Isolation brauchen und wenig Ressourcen verbrauchen. Jeder Service hat ein eigenes Compose-File unter `/opt/<service>/`. Ausnahme: Actual, iSponsorBlockTV, Bambuddy und der Portainer Agent teilen sich `/opt/services/docker-compose.yml` (historisch gewachsen, funktioniert, wird nicht refactored).

## Services

| Service | Port | Compose-Pfad | Funktion |
| --- | --- | --- | --- |
| Actual Budget | 5006 | `/opt/services/` | Budgetverwaltung. Braucht HTTPS für SharedArrayBuffer → Tailscale Serve |
| iSponsorBlockTV | host network | `/opt/services/` | SponsorBlock für Apple TV |
| Bambuddy | 8000 | `/opt/services/` | Bambu Lab A1 Drucker-Monitoring über MQTT |
| Speedtest Tracker | 8765 | `/opt/speedtest-tracker/` | WAN-Speedtests alle 2 Stunden mit Historie und Grafiken |
| Homarr | 7575 | `/opt/homarr/` | Dashboard. Admin-Board und Freunde-Board |
| BetterBahn | 3000 | `/opt/betterbahn/` | DB Split-Ticketing |
| Homebox | 3100 | `/opt/homebox/` | Haushaltsinventar mit QR-Codes |
| Portainer Agent | 9001 | `/opt/services/` | Remote-Management für Portainer auf CT 101 |

### Actual Budget

Braucht SharedArrayBuffer, das nur über HTTPS funktioniert. Gelöst über `tailscale serve --bg --https=5006 http://192.168.178.13:5006` auf dem Proxmox-Host. Zugriff über `https://proxmox.tail4a9632.ts.net:5006`. Die Serve-Config überlebt Reboots automatisch.

### Homarr

Dashboard mit Multi-User-Boards: ein Admin-Board mit allen Services und ein Freunde-Board mit Jellyfin, Seerr, BetterBahn. Proxmox-Integration über API Token (`homarr@pam!monitoring`, PVEAuditor-Rolle, Proxmox CA-Zertifikat hochgeladen).

Extern unter `home.saschafiedler.com` (Cloudflare Tunnel, kein Access davor — Homarr hat eigenen Login).

### BetterBahn

Extern unter `betterbahn.saschafiedler.com`, geschützt mit Cloudflare Access (E-Mail-OTP). Ohne Access wäre es offen, weil BetterBahn keinen eigenen Login hat. Risiko ohne Schutz: Bots könnten die DB-API überlasten und den Zugang für eigene Nutzung blockieren.

## Compose-Files

### /opt/services/docker-compose.yml

```yaml
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

### /opt/speedtest-tracker/docker-compose.yml

```yaml
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

### /opt/homarr/docker-compose.yml

```yaml
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

### /opt/betterbahn/docker-compose.yml

```yaml
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

### /opt/homebox/docker-compose.yml

```yaml
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

Secrets (Speedtest APP_KEY, Homarr SECRET_ENCRYPTION_KEY) liegen in `.env`-Dateien im jeweiligen Verzeichnis.
