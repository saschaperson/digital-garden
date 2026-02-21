---
title: Servarr
publish: false
date: 2026-02-20
tags:
  - homelab
  - media
---

# Servarr

> ⛔ **Dieser Zettel wird nicht veröffentlicht.** Zurück zum [[Homeserver]]-Überblick.

Alles rund um Medienkonsum und -verwaltung: Jellyfin als Mediaserver, Arr-Stack für automatisierte Beschaffung und Organisation.

---

## Jellyfin

Open-Source-Mediaserver, greift auf die Medienbibliothek unter `/mnt/media` zu.

- **Image:** `lscr.io/linuxserver/jellyfin:latest`
- **Port:** `8096`
- **Medien:** `/mnt/media:/media:ro`
- **Transcoding:** `/dev/shm` (RAM) – auf dem Pi aber Direct Play/Stream bevorzugen

### Docker Compose (Jellyfin)

```yaml
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped
    environment:
      PUID: "${PUID}"
      PGID: "${PGID}"
      TZ: ${TZ}
    volumes:
      - ./jellyfin/config:/config
      - /mnt/media:/media:ro
      - /dev/shm:/transcode
    ports:
      - "8096:8096"
```

### Clients

<!-- Welche Geräte/Apps nutzt du? -->

### Troubleshooting Jellyfin

- **Medien werden nicht erkannt:** Dateirechte prüfen (`PUID`/`PGID` passend?)
- **Buffering bei Wiedergabe:** Transcoding vermeiden → Client-Einstellungen auf „Original" / „Maximum"
- **Externe Platte nicht gemountet:** `lsblk` und `mount` prüfen, ggf. fstab-Eintrag

---

## Docker Compose (Servarr-Stack)

```yaml
# Hier den Arr-Stack-Teil deiner Compose ergänzen
```

---

## Services

### SABnzbd

<!-- Config, Pfade, bekannte Probleme (I/O Errors) -->

### Radarr

<!-- Config, Profile, Qualitätseinstellungen -->

### Sonarr

<!-- Config, Profile, Qualitätseinstellungen -->

### Prowlarr

<!-- Indexer-Konfiguration -->

---

## Troubleshooting

<!-- Bekannte Probleme und Lösungen -->
