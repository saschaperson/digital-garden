# Homelab – Media

> CT 103 — Jellyfin als Medienserver. Hardware-Transcoding über die Intel UHD 630 iGPU. Zurück zum [[Homelab]]-Überblick.

---

## Container

| Eigenschaft | Wert |
| --- | --- |
| CT ID | 103 |
| Hostname | `media` |
| IP | 192.168.178.12 |
| Cores | 2 |
| RAM | 2 GB |
| Swap | 512 MB |
| Disk | 20 GB (local-zfs) |
| iGPU | ✅ `dev0: /dev/dri/renderD128,mode=0666` |
| Bind-Mount | `/mnt/storage` → `/mnt/media` (ro) |
| Start Order | 3 |

Read-only Bind-Mount, weil Jellyfin nur liest. Schreibzugriff auf Medien läuft über CT 102 (Servarr).

## Jellyfin

Läuft mit `network_mode: host`, weil die DLNA-Discovery sonst nicht funktioniert und Port-Mapping bei Host-Networking entfällt. Transkodiert über die iGPU (VA-API, Intel UHD 630). Die iGPU wird mit CT 105 (Immich) geteilt.

Transcoding-Buffer liegt auf `/dev/shm` (RAM-Disk), nicht auf der SSD. Spart SSD-Schreibzyklen und ist schneller.

### Libraries

| Library | Pfad im Container | Typ |
| --- | --- | --- |
| Movies | `/mnt/media/Movies` | Movies |
| TV Shows | `/mnt/media/TV Shows` | Shows |
| YouTube | `/mnt/media/YouTube` | Shows |

Die YouTube-Library nutzt den Typ "Shows". Keine aggressiven Remote-Metadata-Provider (TVDB/TMDb) für diese Library, da die lokale Ordnerstruktur wichtiger ist als Metadaten-Matches.

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

## Externer Zugriff

Über Cloudflare Tunnel als `jellyfin.saschafiedler.com`. Kein Cloudflare Access davor, weil Infuse und Swiftfin (iOS/tvOS Jellyfin-Clients) keinen Browser-Login-Flow können. Schutz über Jellyfins eigene Benutzerverwaltung (starke Passwörter, kein Remote-Admin-Zugriff).

## Transcoding prüfen

```bash
# Im LXC:
apt install -y intel-gpu-tools
intel_gpu_top
# Wenn jemand transkodiert, sollten die Render/Video-Balken aktiv sein
```
