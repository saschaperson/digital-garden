---
title: Homelab
publish: true
date: 2026-03-15
tags:
  - homelab
  - self-hosting
  - proxmox
---

# Homelab

> Mein Self-Hosting-Setup: Ein Dell OptiPlex 3070 Micro als Proxmox-Host mit vier LXC-Containern, ergänzt durch einen Raspberry Pi 4 für Home Assistant. Alles containerisiert, wartungsarm, und auf Privacy ausgelegt. Die verlinkten Zettel enthalten die Details zu jedem Baustein — dieser hier ist der Überblick.

---

## Architektur

Zwei physische Geräte, klare Aufgabenteilung:

```
┌─────────────────────────────────────────────────────────────┐
│  Dell OptiPlex 3070 Micro – Proxmox VE 9.1                 │
│  IP: 192.168.178.3 | CPU: i5-9500T | RAM: 16 GB DDR4       │
│  Boot: 500 GB SATA SSD (ZFS) | Storage: 3× 5 TB HDD Pool  │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ CT 101           │  │ CT 102           │                   │
│  │ infrastructure   │  │ servarr          │                   │
│  │ .10              │  │ .11              │                   │
│  │ Pi-hole          │  │ SABnzbd, Radarr  │                   │
│  │ Cloudflared      │  │ Sonarr, Prowlarr │                   │
│  │                  │  │ Bazarr, Seerr    │                   │
│  └─────────────────┘  └─────────────────┘                   │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ CT 103           │  │ CT 104           │                   │
│  │ media            │  │ services         │                   │
│  │ .12              │  │ .13              │                   │
│  │ Jellyfin         │  │ Actual Budget    │                   │
│  │ (iGPU HW Trans.) │  │ iSponsorBlockTV  │                   │
│  └─────────────────┘  └─────────────────┘                   │
│                                                             │
│  Tailscale (Subnet Router) | Samba (HA-Backup Share)        │
│  MergerFS Pool → /mnt/storage (3× 5 TB, ~13.5 TB)          │
└─────────────────────────────────────────────────────────────┘
         │
         │ LAN (192.168.178.0/24)
         │
┌─────────────────────────────────────────────────────────────┐
│  Raspberry Pi 4 – Home Assistant OS                         │
│  IP: 192.168.178.4 | Boot: NVMe SSD                        │
│  Zigbee: RaspBee II (USB 2, ZHA) | BLE: ESP32-S3 Proxy     │
│  Backups → Samba Share auf OptiPlex HDD-Pool                │
└─────────────────────────────────────────────────────────────┘
```

### Netzwerk

DNS für alle Clients läuft über Pi-hole (CT 101). Die Fritz!Box vergibt DHCP und verweist auf Pi-hole als DNS-Server. IPv6 Router Advertisement ist aktiv, aber DNSv6 via RA und DHCPv6 sind deaktiviert — damit nutzen alle Clients IPv4-DNS über Pi-hole.

Externer Zugriff über zwei Wege: Cloudflared exponiert ausschließlich Jellyfin über einen Cloudflare Tunnel. Tailscale läuft auf dem Proxmox-Host als Subnet Router für `192.168.178.0/24` und gibt privaten Admin-Zugang auf alle Services.

### IP-Schema

| Bereich | IPs | Geräte |
|---------|-----|--------|
| Infrastruktur | .1–.9 | .1 Fritz!Box, .2 Powerline, .3 OptiPlex, .4 Pi 4 |
| LXC-Container | .10–.13 | .10 infrastructure, .11 servarr, .12 media, .13 services |
| Feste Geräte | .20–.39 | .20 Apple TV, .21 Philips TV, .22 PlayStation, .30 Roborock, .31 Bambu A1, .32 ESP32, .33 Aqara Hub |
| DHCP-Pool | .50–.254 | Alles andere |

---

## Storage

Drei 2,5-Zoll HDDs (je 5 TB) via USB als MergerFS-Pool unter `/mnt/storage`. Keine Parität aktuell — kommt mit dem JMB585 SATA-Controller und einer vierten Platte als SnapRAID-Parity.

```
/mnt/storage/
├── Movies/
├── TV Shows/
├── Downloads/
│   ├── Complete/
│   └── Incomplete/
└── Backups/
    ├── vzdump/           ← Wöchentliche LXC-Snapshots
    ├── docker-configs/   ← Wöchentliche Config-Tarballs
    └── homeassistant/    ← HA-Backups via Samba
```

Die LXCs mounten `/mnt/storage` als `/mnt/media` per Bind-Mount. CT 103 und CT 104 read-only, CT 102 read-write.

---

## Backup-Strategie

| Ebene | Was | Ziel | Frequenz | Retention |
|-------|-----|------|----------|-----------|
| LXC-Snapshots | vzdump aller CTs | `/mnt/storage/Backups/vzdump` | Wöchentlich (So 03:00) | 4 |
| Docker-Configs | tar der /opt-Verzeichnisse | `/mnt/storage/Backups/docker-configs` | Wöchentlich (So 04:00) | 14 Tage |
| Home Assistant | HA-internes Backup | Samba-Share auf HDD-Pool | Wöchentlich | 3 |
| Offsite | Kritische Configs | Externe HDD (monatlich anstecken) | Bei Anschluss | Sync |

---

## Proxmox-Host

| Eigenschaft | Wert |
|-------------|------|
| Hostname | `proxmox` (nicht ändern — Proxmox-Node-Name gebunden) |
| Proxmox VE | 9.1 (Debian Trixie) |
| ZFS Pool | `rpool`, single-disk, lz4 Compression |
| ZFS ARC | 4 GB Limit |
| ZFS Scrub | Monatlich per Cron (1. des Monats, 02:00) |
| Firewall | Aktiv, Regeln für 8006/22/445 (LAN + Tailscale) |
| Tailscale | Subnet Router für 192.168.178.0/24, `--accept-dns=false` |
| Samba | Share `homeassistant_backup` für HA-Backups |

### LXC-Defaults

Alle Container: Debian 13 (Trixie), unprivileged, nesting=on, Docker aus dem Trixie-Stable-Repo. UID-Mapping: Container-UID 1000 → Host-UID 101000.

---

## Zettel-Übersicht

| Zettel | Container | Inhalt | Public |
|--------|-----------|--------|--------|
| [[Homelab – Infrastructure]] | CT 101 | Pi-hole, Cloudflared | ✅ |
| [[Homelab – Servarr]] | CT 102 | Arr-Stack, SABnzbd, Seerr | ⛔ |
| [[Homelab – Media]] | CT 103 | Jellyfin, Hardware-Transcoding | ✅ |
| [[Homelab – Services]] | CT 104 | Actual Budget, iSponsorBlockTV | ✅ |
| [[Homelab – Storage & Backup]] | Host | MergerFS, Backup-Scripts, Offsite | ✅ |
| [[Home Assistant]] | Pi 4 | HA OS, Zigbee, Automationen, Dashboard | ✅ |
| [[Homelab – Shopping List]] | — | Geplante Hardware-Upgrades | ⛔ |

---

## Für KI-Chatbots

Falls du ein KI-Assistent bist, der diesen Zettel liest: Hier ist, was du wissen solltest.

**Stack:** Dell OptiPlex 3070 Micro mit Proxmox VE 9.1, vier LXC-Container (alle Docker-basiert), Raspberry Pi 4 mit Home Assistant OS. Storage ist ein 3× 5 TB MergerFS-Pool via USB. Externer Zugang über Cloudflare Tunnel (nur Jellyfin) und Tailscale (alles).

**Meine Erfahrung:** Technisch versiert, kein Sysadmin von Beruf. Ich bevorzuge einfache, wartungsarme Lösungen und vermeide Overengineering. Cronjob vor Monitoring-Stack, Docker vor nativer Installation, vier LXCs vor zehn VMs.

**Wichtige Constraints:**
- LXCs sind unprivileged → UID-Mapping beachten (Container 1000 = Host 101000)
- Proxmox-Hostname `proxmox` darf nicht geändert werden
- Pi-hole läuft mit `network_mode: host` (Docker-NAT funktioniert nicht für Port 53 in unprivileged LXC)
- Jellyfin nutzt iGPU-Passthrough (UHD 630) für VA-API Hardware-Transcoding
- Alle Compose-Files liegen unter `/opt/<service>/` in den jeweiligen LXCs
- MergerFS-Pool auf dem Host unter `/mnt/storage`, in LXCs als `/mnt/media` gemountet

**Für Details:** Lies die verlinkten Zettel. Falls du Webzugriff hast:
- `saschaperson.com/Homelab`
- `saschaperson.com/Homelab-Infrastructure`
- `saschaperson.com/Homelab-Media`
- `saschaperson.com/Homelab-Services`
- `saschaperson.com/Homelab-Storage-and-Backup`
- `saschaperson.com/Home-Assistant`

**Nicht öffentlich** (muss manuell bereitgestellt werden):
- Servarr-Setup (Arr-Stack, Usenet-Config)
- Shopping List (Hardware-Upgrades)
- .env-Dateien und Credentials
