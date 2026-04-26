---
title: Homelab
publish: true
date: 2026-04-26
tags:
  - homelab
  - self-hosting
  - proxmox
---

# Homelab

> Mein Self-Hosting-Ökosystem: Ein Dell OptiPlex 3070 Micro als Proxmox-Host, ergänzt durch einen Raspberry Pi 4 für Home Assistant. Externer Zugang über Cloudflare Tunnel und Tailscale. Kalender und Kontakte über Nextcloud auf Uberspace. Alles containerisiert, wartungsarm, auf Datensouveränität ausgelegt.

---

## Architektur

Zwei physische Geräte im Rack, klare Aufgabenteilung:

```
┌─────────────────────────────────────────────────────────────┐
│  Dell OptiPlex 3070 Micro – Proxmox VE                     │
│  CPU: i5-9500T | RAM: 16 GB DDR4                           │
│  Boot: SATA SSD (ZFS) | Storage: 3× 5 TB HDD (MergerFS)   │
│                                                             │
│  CT 101 infrastructure   CT 102 servarr (privat)           │
│  Pi-hole, Cloudflared    SABnzbd, Radarr, Sonarr           │
│  Pulse, Portainer        Prowlarr, Bazarr, Seerr           │
│                          Pinchflat, Recyclarr              │
│                                                             │
│  CT 103 media            CT 104 services                   │
│  Jellyfin                Actual Budget                     │
│  (iGPU HW-Transcoding)   Homarr, BetterBahn               │
│                          Speedtest Tracker, Homebox        │
│                                                             │
│  CT 105 photos           CT 106 documents                  │
│  Immich                  Paperless-ngx                     │
│  (iGPU OpenVINO ML)      (Tika, Gotenberg)                │
│                                                             │
│  Tailscale Subnet Router | Samba (HA-Backup Share)         │
│  MergerFS Pool ~13,5 TB                                    │
└─────────────────────────────────────────────────────────────┘
         │ LAN
┌─────────────────────────────────────────────────────────────┐
│  Raspberry Pi 4 – Home Assistant OS                        │
│  Boot: NVMe SSD (via HAT) | Zigbee: RaspBee II (ZHA)      │
│  BLE: ESP32-S3 Proxy                                       │
└─────────────────────────────────────────────────────────────┘
```

Zugang von außen über zwei Wege: Cloudflare Tunnel für ausgewählte Services, Tailscale für alles andere. Details in [[Homelab – Netzwerk & Zugang]].

---

## Container-Ressourcen

| CT | Name | Cores | RAM | Swap | Disk | iGPU | Public |
|----|------|-------|-----|------|------|------|--------|
| 101 | infrastructure | 2 | 1 GB | 512 MB | 8 GB | — | — |
| 102 | servarr | 2 | 2 GB | 512 MB | 10 GB | — | — |
| 103 | media | 2 | 2 GB | 512 MB | 20 GB | ✅ | Tunnel |
| 104 | services | 2 | 2 GB | 512 MB | 12 GB | — | Tunnel/Tailscale |
| 105 | photos | 4 | 4 GB | 1 GB | 16 GB | ✅ | Tailscale |
| 106 | documents | 2 | 2 GB | 512 MB | 12 GB | — | Tailscale |

Alle Container: Debian Trixie, unprivileged, Docker aus dem offiziellen Docker-Repo.

---

## Proxmox-Host

| Eigenschaft | Wert |
|-------------|------|
| Proxmox VE | aktuell |
| ZFS Pool | `rpool`, single-disk, lz4 Compression |
| ZFS ARC | 2 GB Limit |
| Swap | 4 GB ZFS zvol, swappiness=10 |
| Tailscale | Subnet Router für LAN, `--accept-dns=false` |
| Samba | Share `homeassistant_backup` für HA-Backups |

### LXC-Defaults

Alle Container unprivileged mit nesting=on. **UID-Mapping:** Container-UID 0 (root) → Host-UID 100000. Container-UID 1000 → Host-UID 101000. Bind-Mount-Ordner müssen entsprechend `chown` bekommen.

**iGPU-Passthrough:** CT 103 und CT 105 teilen sich die Intel UHD 630 über `dev0: /dev/dri/renderD128,mode=0666`. Kein exklusiver Passthrough.

---

## Zettel-Übersicht

| Zettel | Inhalt |
|--------|--------|
| [[Homelab – Netzwerk & Zugang]] | Fritz!Box, Tailscale, Cloudflare Zero Trust |
| [[Homelab – Infrastructure]] | CT 101: Pi-hole, Cloudflared, Pulse, Portainer |
| [[Homelab – Media]] | CT 103: Jellyfin, Hardware-Transcoding |
| [[Homelab – Services]] | CT 104: Actual, Homarr, BetterBahn, Homebox |
| [[Homelab – Photos]] | CT 105: Immich, OpenVINO ML |
| [[Homelab – Documents]] | CT 106: Paperless-ngx |
| [[Homelab – Storage & Backup]] | MergerFS, Backup-Strategie |
| [[Home Assistant]] | Pi 4: HA OS, Zigbee, Automationen, iPad Dashboard |
| [[Nextcloud]] | CalDAV/CardDAV auf Uberspace, HA-Integration |
| [[Mein Homelab]] | Persönlicher Hintergrund, Hardware, Rack |

---

## Für KI-Assistenten

Falls du ein KI-Assistent bist, der dieses Dokument liest — hier ist, was du wissen musst.

**Was bereits läuft — bitte nicht erneut vorschlagen:**

| Kategorie | Service |
|-----------|---------|
| Medien | Jellyfin, Immich, Servarr-Stack (Radarr, Sonarr, Prowlarr, SABnzbd), Pinchflat |
| Dokumente | Paperless-ngx |
| Fotos | Immich |
| Smart Home | Home Assistant (Pi 4, extern) |
| Kalender/Kontakte | Nextcloud auf Uberspace |
| Finanzen | Actual Budget |
| DNS & Blocking | Pi-hole |
| Monitoring | Pulse (Proxmox), Speedtest Tracker |
| Dashboard | Homarr |
| Inventar | Homebox |
| Bahn | BetterBahn |

**Zugangsschicht:** Cloudflare Tunnel + Tailscale. Kein klassischer Reverse Proxy. Kein Port-Forwarding. Externer Zugriff läuft ausschließlich über diese beiden Wege.

**Identifiers:** IPs, Ports, Subdomains, Tailnet-Name und Credentials liegen in `Homelab – Privat.md` (nicht öffentlich). Ohne diese Datei im Prompt-Kontext sind konkrete Netzwerkadressen unbekannt.

**Philosophie:** Wartungsarm vor feature-reich. Cronjob vor Monitoring-Stack. Docker vor nativer Installation. Sechs LXCs vor zehn VMs. Neue Services bekommen eigene Compose-Files unter `/opt/<service>/`.

---

## Dokumentations-Prinzipien

Dieser Zettel ist als **einzelne URL für Prompt-Kontext** konzipiert — `saschaperson.com/Homelab` rendert alle Unter-Seiten per Transklusion auf einer Seite. Einem KI-Assistenten genügt diese URL plus `Homelab – Privat.md` als Kontext.

**Strukturregeln für zukünftige Dokumentation:**
- Konzepte, Entscheidungslogik und Compose-Patterns gehören in die öffentlichen Zettel
- Alle Identifier (IPs, Ports, Subdomains, Tokens, Usernamen) gehören ausschließlich in `Homelab – Privat.md`
- Jeder LXC bekommt einen eigenen Zettel — kein Zusammenfassen
- Hardware-unabhängige Services (HA, Nextcloud) bekommen eigene Zettel ohne Homelab-Prefix
- Keine vergangene Konfiguration dokumentieren — nur aktueller Stand und Entscheidungsgründe

---

![[Homelab – Netzwerk & Zugang]]
![[Homelab – Infrastructure]]
![[Homelab – Media]]
![[Homelab – Services]]
![[Homelab – Photos]]
![[Homelab – Documents]]
![[Homelab – Storage & Backup]]
![[Home Assistant]]
![[Nextcloud]]
