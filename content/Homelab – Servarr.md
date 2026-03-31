---
title: Homelab – Servarr
publish: false
tags:
  - homelab
  - media
  - servarr
  - pinchflat
  - jellyfin
---
> ⛔ **Nicht veröffentlichen.** CT 102 auf dem Proxmox-Host. Arr-Stack für automatisierte Medienverwaltung plus YouTube-Ingest via Pinchflat. Zurück zum [[Homelab]]-Überblick.

---

## Container

| Eigenschaft | Wert |
|---|---|
| CT ID | 102 |
| Hostname | `servarr` |
| IP | 192.168.178.11 |
| Cores | 2 |
| RAM | 2 GB |
| Swap | 512 MB |
| Disk | 8 GB (local-zfs, ~4.8 GB belegt — bei Docker-Updates im Auge behalten) |
| DNS | 192.168.178.10 (Pi-hole) |
| Bind-Mount | `/mnt/storage` → `/mnt/media` (rw) |
| Start Order | 2, up=30 |
| Compose | `/opt/servarr/docker-compose.yml` |

Disk wurde von 120 GB auf 8 GB per Backup/Restore geschrumpft. Bei über 80% Belegung (`docker system df`) auf 12 GB vergrößern — ZFS kann vergrößern ohne Destroy.

Der Bind-Mount ging beim Shrink-Restore verloren und musste manuell wiederhergestellt werden. Bei zukünftigen Restores immer prüfen, ob `mp0` in der LXC-Config steht.

Kein separates `.env`-File. Variablen inline in der Compose. Standard-User: `PUID=1000`, `PGID=1000`.

## Services

| Service | Port | Funktion |
|---|---:|---|
| SABnzbd | 8082 | Usenet-Downloader |
| Prowlarr | 9696 | Indexer-Manager |
| Radarr | 7878 | Filme |
| Sonarr | 8989 | Serien |
| Bazarr | 6767 | Untertitel |
| Seerr | 5055 | Request-Frontend (extern via Tunnel) |
| Recyclarr | — | TRaSH-Guide-Sync (Cron, kein Web-UI) |
| Pinchflat | 8945 | YouTube-/Playlist-Ingest für Jellyfin |
| Portainer Agent | 9001 | Remote-Management |

## Compose

```yaml
# /opt/servarr/docker-compose.yml
services:
  sabnzbd:
    image: lscr.io/linuxserver/sabnzbd:latest
    container_name: sabnzbd
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
    volumes:
      - ./sabnzbd:/config
      - /mnt/media/Downloads/Complete:/downloads
      - /mnt/media/Downloads/Incomplete:/incomplete-downloads
    ports:
      - "8082:8080"
    restart: unless-stopped

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
    volumes:
      - ./prowlarr:/config
    ports:
      - "9696:9696"
    restart: unless-stopped

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
    volumes:
      - ./radarr:/config
      - /mnt/media:/media
    ports:
      - "7878:7878"
    restart: unless-stopped

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
    volumes:
      - ./sonarr:/config
      - /mnt/media:/media
    ports:
      - "8989:8989"
    restart: unless-stopped

  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
    volumes:
      - ./bazarr:/config
      - /mnt/media:/media
    ports:
      - "6767:6767"
    restart: unless-stopped

  seerr:
    image: ghcr.io/seerr-team/seerr:latest
    init: true
    container_name: seerr
    environment:
      - TZ=Europe/Berlin
      - PORT=5055
    ports:
      - "5055:5055"
    volumes:
      - ./seerr:/app/config
    restart: unless-stopped

  recyclarr:
    image: ghcr.io/recyclarr/recyclarr:8
    container_name: recyclarr
    user: 1000:1000
    environment:
      - TZ=Europe/Berlin
      - RECYCLARR_CREATE_CONFIG=true
    volumes:
      - /opt/servarr/recyclarr:/config
    restart: unless-stopped

  pinchflat:
    image: ghcr.io/kieraneglin/pinchflat:latest
    container_name: pinchflat
    user: "1000:1000"
    restart: unless-stopped
    environment:
      - TZ=Europe/Berlin
      - LOG_LEVEL=info
    ports:
      - "8945:8945"
    volumes:
      - /opt/servarr/pinchflat:/config
      - /mnt/media/YouTube:/downloads

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

---

## Seerr-Anbindung

| Eigenschaft | Wert |
|---|---|
| Jellyfin | `http://192.168.178.12:8096` |
| Externer Zugang | `https://seerr.saschafiedler.com` (Tunnel, eigener Login) |
| Radarr | `http://192.168.178.11:7878`, Profil `[German] HD Bluray + WEB`, Ziel `/media/Movies` |
| Sonarr | `http://192.168.178.11:8989`, Profil `[German] HD Bluray + WEB`, Ziel `/media/TV Shows` |

Seerr authentifiziert gegen Jellyfin. Wenn die Jellyfin-Verbindung nicht steht, kann sich niemand einloggen. Die Jellyfin-IP steht in der Seerr-DB (`/opt/servarr/seerr/db/db.sqlite3`), nicht nur in der `settings.json`.

---

## Pfade & Volumes

Import in Radarr/Sonarr ist Copy, nicht Hardlink. Die Container-Volumes sind als separate Bind-Mounts konfiguriert, was Hardlinks verhindert. Für ein zukünftiges Refactoring wäre ein einzelner `/data`-Mount nach TRaSH-Best-Practice besser.

| Service | Container-Pfad | Host-Pfad (LXC) |
|---|---|---|
| SABnzbd | `/downloads` | `/mnt/media/Downloads/Complete` |
| SABnzbd | `/incomplete-downloads` | `/mnt/media/Downloads/Incomplete` |
| Radarr/Sonarr | `/media` | `/mnt/media` |
| Pinchflat | `/config` | `/opt/servarr/pinchflat` |
| Pinchflat | `/downloads` | `/mnt/media/YouTube` |

### Docker-interne Kommunikation

Alle Services im selben Compose-Netzwerk, daher funktionieren Container-Namen als Hostnamen:

| Von → Nach | Wert |
|---|---|
| Radarr/Sonarr → SABnzbd | `sabnzbd:8080` |
| Prowlarr → Radarr | `http://radarr:7878` |
| Prowlarr → Sonarr | `http://sonarr:8989` |

Seerr → Jellyfin nutzt die IP `192.168.178.12:8096` (anderer LXC, nicht im selben Docker-Netzwerk).

---

## SABnzbd

| Einstellung | Wert |
|---|---|
| Kategorie-Processing | +Delete (Default, movies, tv) |
| Encrypted RAR | Abort |
| Direct Unpack | ✅ |
| Recursive Unpacking | ✅ |
| Deobfuscate Filenames | ✅ |

Usenet-Server: `news.eweka.nl:563` (SSL) und `news.usenetexpress.com:563` (SSL).

---

## Recyclarr & Qualitätsprofile

Profile via Recyclarr aus den TRaSH Guides synchronisiert. Config: `/opt/servarr/recyclarr/recyclarr.yml`, Secrets: `/opt/servarr/recyclarr/secrets.yml`.

Wichtig: `delete_old_custom_formats: true` und `reset_unmatched_scores: enabled: true`. Manuelle CF-Änderungen in Sonarr/Radarr werden beim nächsten Sync überschrieben. Score-Overrides müssen in die `recyclarr.yml`.

### Radarr-Profile

| Profil | Verwendung |
|---|---|
| [German] HD Bluray + WEB | Standard — deutsche Tonspur, 720p–1080p |
| [German] UHD Bluray + WEB | Favoriten — deutsche Tonspur, 4K |
| HD Bluray + WEB | Fallback — jede Sprache, 1080p |
| Any | Catch-All |

### Score-Logik (German-Profil)

German DL (Dual Language) mit +11.000 stark bevorzugt. HEVC mit +200 Override leicht bevorzugt (alle Clients unterstützen HEVC nativ). German Microsized und Not German or English mit -35.000 blockiert. Naming: `jellyfin-tmdb` (Radarr), `jellyfin-tvdb` (Sonarr).

### Recyclarr-Befehle

```bash
cd /opt/servarr
docker compose run --rm recyclarr sync          # Manueller Sync
docker compose run --rm recyclarr sync --preview # Preview ohne Änderungen
```

---

## Pinchflat

YouTube-Ingest für Jellyfin. Source-Namen so wählen, wie der Ordner in Jellyfin heißen soll. SponsorBlock-Kategorien: sponsor, selfpromo, interaction, intro, outro.

| Pfad | Zweck |
|---|---|
| `/opt/servarr/pinchflat/extras/cookies.txt` | YouTube-Cookies (nur wenn nötig) |
| `/opt/servarr/pinchflat/extras/yt-dlp-configs/base-config.txt` | Globale yt-dlp-Config |

Aktuell steht `--ignore-errors` in der base-config als Workaround für Subtitle-Fehler. Pragmatisch, nicht ideal — kann andere Fehler schlucken. Bei merkwürdigem Verhalten zuerst dort prüfen.

Jellyfin YouTube-Library: Typ "Shows", Root `/mnt/media/YouTube`. Keine aggressiven Remote-Metadata-Provider.

---

## Datenfluss

```
Seerr (Anfrage)
  ├─→ Radarr → Prowlarr → SABnzbd → /Downloads/Complete → Radarr import (Copy) → /Movies
  └─→ Sonarr → Prowlarr → SABnzbd → /Downloads/Complete → Sonarr import (Copy) → /TV Shows

Pinchflat → /YouTube/<Source-Name>/Season 01/...

Jellyfin (CT 103, read-only) ←── Movies, TV Shows, YouTube
Infuse ←── Jellyfin
```

---

## Troubleshooting

- **Service nicht erreichbar:** `docker ps -a`, dann `docker logs <service> --tail 50`
- **Berechtigungen auf `/mnt/media`:** Container-UID 1000 = Host-UID 101000. Dateien auf dem Host unter `/mnt/storage/` bearbeiten oder per `docker exec --user 1000:1000`
- **SABnzbd startet nicht nach Reboot:** Bind-Mount prüfen — `pct config 102 | grep mp0`. Wenn fehlend: `pct set 102 --mp0 /mnt/storage,mp=/mnt/media`
- **Seerr Login geht nicht:** Jellyfin-Verbindung prüfen. Seerr authentifiziert gegen Jellyfin. IP in settings.json muss `192.168.178.12` sein, nicht `jellyfin`
- **Pinchflat restartet:** Berechtigungen auf `/mnt/storage/YouTube` (Host: `chown -R 101000:101000`) und `/opt/servarr/pinchflat` (LXC: `chown -R 1000:1000`)
- **Recyclarr Permission Denied:** `chown -R 1000:1000 /opt/servarr/recyclarr`
- **Doppelte Jahreszahl in Serienordnern:** Manuell umbenennen + Pfad in Sonarr anpassen. Bekanntes Problem wenn TVDB-Name bereits Jahreszahl enthält
- **Disk über 80%:** `docker system prune` oder auf 12 GB vergrößern (ZFS: ohne Destroy möglich)
