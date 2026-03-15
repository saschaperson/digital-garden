---
title: Homelab – Servarr
publish: false
date: 2026-03-15
tags:
  - homelab
  - media
  - servarr
---

# Homelab – Servarr

> ⛔ **Nicht veröffentlichen.** CT 102 auf dem Proxmox-Host. Arr-Stack für automatisierte Medienverwaltung. Zurück zum [[Homelab]]-Überblick.

---

## Container-Konfiguration

| Eigenschaft | Wert |
|-------------|------|
| CT ID | 102 |
| Hostname | `servarr` |
| IP | 192.168.178.11 |
| RAM | 2048 MB |
| Disk | 120 GB (ZFS Subvolume, ~3 GB belegt) |
| Cores | (Proxmox Default) |
| Mount | `/mnt/storage` → `/mnt/media` (read-write) |

### Compose & Secrets

- Compose: `/opt/servarr/docker-compose.yml`
- Kein separates `.env`-File — Variablen inline in der Compose
- PUID/PGID: `1000`/`1000`
- Auth: Forms (alle Services)

---

## Services

| Service | Port | Container-Port | Funktion |
|---------|------|----------------|----------|
| SABnzbd | 8082 | 8080 | Usenet-Downloader |
| Prowlarr | 9696 | 9696 | Indexer-Manager |
| Radarr | 7878 | 7878 | Filme |
| Sonarr | 8989 | 8989 | Serien |
| Bazarr | 6767 | 6767 | Untertitel |
| Seerr | 5055 | 5055 | Request-Frontend |

### Pfade

| Service | Einstellung | Wert |
|---------|-------------|------|
| SABnzbd | Completed Downloads | `/downloads` (→ `/mnt/media/Downloads/Complete`) |
| SABnzbd | Incomplete Downloads | `/incomplete-downloads` (→ `/mnt/media/Downloads/Incomplete`) |
| Radarr | Root Folder | `/media/Movies` |
| Sonarr | Root Folder | `/media/TV Shows` |
| Radarr/Sonarr | Download Client Host | `sabnzbd` (Docker-intern) |
| Radarr/Sonarr | Download Client Port | `8080` |
| Prowlarr | Radarr URL | `http://radarr:7878` |
| Prowlarr | Sonarr URL | `http://sonarr:8989` |

### Usenet-Server

| Server | Port | SSL |
|--------|------|-----|
| `news.eweka.nl` | 563 | ✅ |
| `news.usenetexpress.com` | 563 | ✅ |

### Qualitätsprofile

Radarr und Sonarr nutzen das Profil "UHD German" (Profil-ID 7), konfiguriert via Seerr.

---

## Datenfluss

```
Seerr (Anfrage)
  ├──→ Radarr (Filme) ──→ Prowlarr (Indexer) ──→ SABnzbd (Download)
  │                                                    ↓
  │                                        /mnt/media/Downloads/Complete
  │                                                    ↓
  │                                        Radarr importiert → /mnt/media/Movies
  │
  └──→ Sonarr (Serien) ──→ Prowlarr (Indexer) ──→ SABnzbd (Download)
                                                       ↓
                                           /mnt/media/Downloads/Complete
                                                       ↓
                                           Sonarr importiert → /mnt/media/TV Shows

Jellyfin (CT 103) ←── /mnt/media (read-only)
Bazarr ←── Radarr/Sonarr API → lädt Untertitel nach
```

---

## Seerr-Anbindung

| Eigenschaft | Wert |
|-------------|------|
| Media-Server | Jellyfin (IP: 192.168.178.12, Port: 8096) |
| Externer Zugang | `https://jellyfin.saschafiedler.com` |
| Radarr | `http://192.168.178.11:7878`, Profil "UHD German", Ziel `/media/Movies` |
| Sonarr | `http://192.168.178.11:8989`, Profil "UHD German", Ziel `/media/TV Shows` |

---

## Troubleshooting

- **SABnzbd startet nicht:** Prüfe ob `/mnt/media/Downloads/Complete` und `/mnt/media/Downloads/Incomplete` existieren und UID 101000:101000 auf dem Host haben
- **Import schlägt fehl (Radarr/Sonarr):** Berechtigungen prüfen — `/mnt/media` muss read-write gemountet sein, Dateien müssen UID 1000 innerhalb der LXC haben
- **Service nicht erreichbar:** `docker ps -a` in der LXC, `docker logs <service> --tail 20`
- **Config-Migration von altem Setup:** Configs nach `/opt/servarr/<service>/` kopieren, `chown -R 1000:1000` innerhalb der LXC (nicht auf dem Host — UID-Mapping!)
