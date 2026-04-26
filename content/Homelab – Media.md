---
title: Homelab – Media
publish: true
date: 2026-04-26
tags:
  - homelab
  - jellyfin
  - media
---

# Homelab – Media

> CT 103 — Jellyfin als Medienserver. Hardware-Transcoding über die Intel UHD 630 iGPU. Zurück zum [[Homelab]]-Überblick.

---

## Container

| Eigenschaft | Wert |
|-------------|------|
| CT ID | 103 |
| Hostname | `media` |
| Cores | 2 |
| RAM | 2 GB |
| Swap | 512 MB |
| Disk | 20 GB (local-zfs) |
| iGPU | ✅ `dev0: /dev/dri/renderD128,mode=0666` |
| Bind-Mount | `/mnt/storage` → `/mnt/media` (ro) |
| Start Order | 3 |

Read-only Bind-Mount — Jellyfin liest nur. Schreibzugriff auf Medien läuft über CT 102 (Servarr).

---

## Jellyfin

Läuft mit `network_mode: host` wegen DLNA-Discovery. Transkodiert über die iGPU via VA-API. Die iGPU wird mit CT 105 (Immich) geteilt.

Transcoding-Buffer auf `/dev/shm` (RAM-Disk) statt SSD — spart Schreibzyklen und ist schneller.

**Externer Zugang:** Cloudflare Tunnel. Kein Cloudflare Access davor — Infuse und Swiftfin (iOS/tvOS-Clients) können keinen Browser-Login-Flow. Schutz über Jellyfins eigene Benutzerverwaltung.

### Libraries

| Library | Pfad im Container | Typ |
|---------|-------------------|-----|
| Movies | `/mnt/media/Movies` | Movies |
| TV Shows | `/mnt/media/TV Shows` | Shows |
| YouTube | `/mnt/media/YouTube` | Shows |

Die YouTube-Library nutzt den Typ „Shows". Keine aggressiven Remote-Metadata-Provider für diese Library — lokale Ordnerstruktur hat Vorrang.

---

## Compose

```yaml
# /opt/media/docker-compose.yml
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
    volumes:
      - ./jellyfin/config:/config
      - /mnt/media:/media:ro
      - /dev/shm:/transcode
    devices:
      - /dev/dri/renderD128:/dev/dri/renderD128
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

`card0` wird nicht durchgereicht — Jellyfin braucht nur `renderD128` für VA-API Transcoding.

---

## Transcoding prüfen

```bash
apt install -y intel-gpu-tools
intel_gpu_top
# Render/Video-Balken sollten bei aktivem Transcoding ausschlagen
```
