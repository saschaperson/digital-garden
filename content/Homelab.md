# Homelab

> Mein Self-Hosting-Setup: Ein Dell OptiPlex 3070 Micro als Proxmox-Host mit sechs LXC-Containern, ergänzt durch einen Raspberry Pi 4 für Home Assistant. Alles containerisiert, wartungsarm, und auf Privacy ausgelegt. Die verlinkten Zettel enthalten die Details zu jedem Baustein — dieser hier ist der Überblick.

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
│  │ CT 101           │  │ CT 102           │                  │
│  │ infrastructure   │  │ servarr          │                  │
│  │ .10              │  │ .11              │                  │
│  │ Pi-hole          │  │ SABnzbd, Radarr  │                  │
│  │ Cloudflared      │  │ Sonarr, Prowlarr │                  │
│  │ Pulse, Portainer │  │ Bazarr, Seerr    │                  │
│  │                  │  │ Recyclarr        │                  │
│  │                  │  │ Pinchflat        │                  │
│  └─────────────────┘  └─────────────────┘                   │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ CT 103           │  │ CT 104           │                  │
│  │ media            │  │ services         │                  │
│  │ .12              │  │ .13              │                  │
│  │ Jellyfin         │  │ Actual Budget    │                  │
│  │ (iGPU HW Trans.) │  │ iSponsorBlockTV  │                  │
│  │                  │  │ Speedtest Tracker│                  │
│  │                  │  │ Homarr, BetterBahn│                 │
│  │                  │  │ Homebox, Bambuddy│                  │
│  └─────────────────┘  └─────────────────┘                   │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ CT 105           │  │ CT 106           │                  │
│  │ photos           │  │ documents        │                  │
│  │ .14              │  │ .15              │                  │
│  │ Immich           │  │ Paperless-ngx    │                  │
│  │ (iGPU OpenVINO)  │  │ (Tika, Gotenberg)│                 │
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
│  IP: 192.168.178.4 | Boot: NVMe SSD (via HAT, USB-3-Bridge)│
│  Zigbee: RaspBee II (USB 2, ZHA) | BLE: ESP32-S3 Proxy     │
│  Backups → Samba Share auf OptiPlex HDD-Pool                │
└─────────────────────────────────────────────────────────────┘
```

### Netzwerk

DNS für alle Clients läuft über Pi-hole (CT 101). Die Fritz!Box vergibt DHCP und verweist auf Pi-hole als DNS-Server. IPv6 Router Advertisement ist aktiv, aber DNSv6 via RA und DHCPv6 sind deaktiviert.

Externer Zugriff über zwei Wege:
- **Cloudflare Tunnel** exponiert vier Services über Subdomains. BetterBahn ist hinter Cloudflare Access mit E-Mail-OTP geschützt, die anderen haben eigene Authentifizierung.
- **Tailscale** läuft auf dem Proxmox-Host als Subnet Router für `192.168.178.0/24`. Gibt Zugriff auf alle Services von unterwegs. Tailscale Serve stellt Actual Budget unter `https://proxmox.tail4a9632.ts.net:5006` mit gültigem TLS-Zertifikat bereit (nötig wegen SharedArrayBuffer).

#### Cloudflare Tunnel Subdomains

| Subdomain | Service | CT | Absicherung |
| --- | --- | --- | --- |
| jellyfin.saschafiedler.com | Jellyfin | 103 | Jellyfin-eigener Login (kein Access — Infuse/Swiftfin kann keinen Browser-Login) |
| seerr.saschafiedler.com | Seerr | 102 | Seerr-eigener Login mit Benutzerverwaltung |
| betterbahn.saschafiedler.com | BetterBahn | 104 | Cloudflare Access (E-Mail-OTP, kein eigener Login) |
| home.saschafiedler.com | Homarr | 104 | Homarr-eigener Login (Multi-User Boards) |

Cloudflare Access Cookie-Domain ist `.saschafiedler.com` — ein OTP-Login gilt für alle Access-geschützten Subdomains.

### IP-Schema

| Bereich | IPs | Geräte |
| --- | --- | --- |
| Infrastruktur | .1–.9 | .1 Fritz!Box, .2 Powerline, .3 OptiPlex, .4 Pi 4 |
| LXC-Container | .10–.15 | .10 infrastructure, .11 servarr, .12 media, .13 services, .14 photos, .15 documents |
| Feste Geräte | .20–.39 | .20 Apple TV, .21 Philips TV, .22 PlayStation, .30 Roborock, .31 Bambu A1, .32 ESP32, .33 Aqara Hub |
| DHCP-Pool | .50–.254 | Alles andere |

### Port-Übersicht

| CT | Service | Port | Extern |
| --- | --- | --- | --- |
| 101 | Pi-hole Admin | 80 | Nein |
| 101 | Portainer | 9443 | Nein |
| 101 | Pulse | 7655 | Nein |
| 102 | SABnzbd | 8082 | Nein |
| 102 | Sonarr | 8989 | Nein |
| 102 | Radarr | 7878 | Nein |
| 102 | Prowlarr | 9696 | Nein |
| 102 | Bazarr | 6767 | Nein |
| 102 | Seerr | 5055 | Tunnel |
| 102 | Pinchflat | 8945 | Nein |
| 103 | Jellyfin | 8096 | Tunnel |
| 104 | Actual Budget | 5006 | Tailscale Serve (HTTPS) |
| 104 | Speedtest Tracker | 8765 | Nein |
| 104 | Homarr | 7575 | Tunnel |
| 104 | BetterBahn | 3000 | Tunnel (Access) |
| 104 | Homebox | 3100 | Nein |
| 104 | Bambuddy | 8000 | Nein |
| 105 | Immich | 2283 | Tailscale |
| 106 | Paperless-ngx | 8000 | Tailscale |

---

## Container-Ressourcen

| CT | Name | Cores | RAM | Swap | Disk | iGPU |
| --- | --- | --- | --- | --- | --- | --- |
| 101 | infrastructure | 2 | 1 GB | 512 MB | 4 GB | — |
| 102 | servarr | 2 | 2 GB | 512 MB | 8 GB | — |
| 103 | media | 2 | 2 GB | 512 MB | 20 GB | ✅ |
| 104 | services | 2 | 2 GB | 512 MB | 12 GB | — |
| 105 | photos | 4 | 4 GB | 1 GB | 16 GB | ✅ |
| 106 | documents | 2 | 2 GB | 512 MB | 12 GB | — |

### Compose-Struktur

Historisch gewachsen gibt es zwei Muster:
- **Ein Compose pro LXC:** CT 102 (`/opt/servarr/`), CT 103 (`/opt/media/`), CT 104 (`/opt/services/` für ältere Services)
- **Ein Compose pro Service:** Pulse, Homarr, Speedtest Tracker, BetterBahn, Homebox, Immich, Paperless

Separate Composes sind besser (isolierte Updates, kleinerer Blast Radius bei Problemen), aber die bestehenden zusammengefassten Composes laufen stabil und werden nicht refactored. Neue Services bekommen immer eigene Composes unter `/opt/<service>/`.

Secrets liegen in `.env`-Dateien im jeweiligen Compose-Verzeichnis, nicht hardcodiert in den Compose-Files. So können die Composes öffentlich dokumentiert werden.

---

## Proxmox-Host

| Eigenschaft | Wert |
| --- | --- |
| Hostname | `proxmox` (nicht ändern — Proxmox-Node-Name gebunden) |
| Proxmox VE | 9.1 (Debian Trixie) |
| ZFS Pool | `rpool`, single-disk, lz4 Compression |
| ZFS ARC | 2 GB Limit (`/etc/modprobe.d/zfs.conf`) |
| ZFS Auto-TRIM | on |
| ZFS Scrub | Monatlich per Cron (1. des Monats, 02:00) |
| Swap | 4 GB ZFS zvol (`rpool/swap`), swappiness=10 |
| Firewall | Aktiv, Regeln für 8006/22/445 (LAN + Tailscale) |
| Tailscale | Subnet Router für 192.168.178.0/24, `--accept-dns=false` |
| Tailscale Serve | `https://proxmox.tail4a9632.ts.net:5006` → Actual Budget |
| Samba | Share `homeassistant_backup` für HA-Backups |
| Monitoring | Pulse (via API Token `pulse@pam!monitoring`) + Homarr (via `homarr@pam!monitoring`) |

### LXC-Defaults

Alle Container: Debian 13 (Trixie), unprivileged, nesting=on, Docker aus dem offiziellen Docker-Repo (nicht Debian). Locale: en_US.UTF-8 auf allen Containern gesetzt.

**UID-Mapping:** Container-UID 0 (root) → Host-UID 100000. Container-UID 1000 → Host-UID 101000. Bind-Mount-Ordner auf dem Host müssen `chown` auf die korrekte Host-UID bekommen, je nachdem welcher User im Container schreibt. Immich läuft als root (→ 100000), fast alles andere als User 1000 (→ 101000).

**iGPU-Passthrough:** Beide Container (CT 103, CT 105) nutzen die `dev0: /dev/dri/renderD128,mode=0666` Methode in der LXC-Config. Die iGPU (Intel UHD 630) wird geteilt — kein exklusiver Passthrough.

---

## Zettel-Übersicht

| Zettel | Container | Inhalt | Public |
| --- | --- | --- | --- |
| [[Homelab – Infrastructure]] | CT 101 | Pi-hole, Cloudflared, Pulse, Portainer | ✅ |
| [[Homelab – Servarr]] | CT 102 | Arr-Stack, SABnzbd, Seerr, Pinchflat | ⛔ |
| [[Homelab – Media]] | CT 103 | Jellyfin, Hardware-Transcoding | ✅ |
| [[Homelab – Services]] | CT 104 | Actual, Speedtest, Homarr, BetterBahn, Homebox, Bambuddy | ✅ |
| [[Homelab – Photos]] | CT 105 | Immich, OpenVINO ML, VAAPI Transcoding | ✅ |
| [[Homelab – Documents]] | CT 106 | Paperless-ngx, Tika, Gotenberg | ✅ |
| [[Homelab – Storage & Backup]] | Host | MergerFS, Backup-Scripts, Boot-Fix, Offsite | ✅ |
| [[Home Assistant]] | Pi 4 | HA OS, Zigbee, Automationen, Dashboard | ✅ |
| [[Mein Homelab]] | — | Persönlicher Hintergrund, Hardware, Rack | ✅ |
| [[Homelab – Shopping List]] | — | Geplante Hardware-Upgrades | ⛔ |

---

## Für KI-Chatbots

Falls du ein KI-Assistent bist, der diesen Zettel liest: Hier ist, was du wissen solltest.

**Stack:** Dell OptiPlex 3070 Micro mit Proxmox VE 9.1, sechs LXC-Container (alle Docker-basiert), Raspberry Pi 4 mit Home Assistant OS. Storage ist ein 3× 5 TB MergerFS-Pool via USB. Externer Zugang über Cloudflare Tunnel (Jellyfin, Seerr, BetterBahn, Homarr) und Tailscale (alles).

**Meine Erfahrung:** Technisch versiert, kein Sysadmin von Beruf. Ich bevorzuge einfache, wartungsarme Lösungen und vermeide Overengineering. Cronjob vor Monitoring-Stack, Docker vor nativer Installation, sechs LXCs vor zehn VMs.

**Wichtige Constraints:**

* LXCs sind unprivileged → UID-Mapping beachten (Container 0 = Host 100000, Container 1000 = Host 101000)
* Bind-Mount-Ordner auf dem Host müssen `chown` auf die korrekte Host-UID
* Proxmox-Hostname `proxmox` darf nicht geändert werden
* Pi-hole läuft mit `network_mode: host` (Docker-NAT funktioniert nicht für Port 53 in unprivileged LXC)
* Jellyfin und Immich teilen sich die iGPU (UHD 630) über `dev0:` Passthrough
* Compose-Files liegen unter `/opt/<service>/` oder `/opt/<lxc-name>/` in den LXCs
* Secrets gehören in `.env`-Dateien, nicht in docker-compose.yml
* MergerFS-Pool auf dem Host unter `/mnt/storage`, in LXCs als `/mnt/media`, `/mnt/photos` oder `/mnt/documents`
* MergerFS Boot-Fix: fstab hat `x-systemd.device-timeout=120` und `x-systemd.requires/after` für die Disks. `pve-guests.service` wartet über ein Drop-in auf `mnt-storage.mount`
* 16 GB RAM ist das aktuelle Limit — ZFS ARC auf 2 GB reduziert, 4 GB Swap als Puffer. 32 GB Upgrade geplant

**Für Details:** Lies die verlinkten Zettel. Falls du Webzugriff hast: `saschaperson.com/Homelab` und die verlinkten Unterseiten.

**Nicht öffentlich** (muss manuell bereitgestellt werden): Servarr-Setup, Shopping List, .env-Dateien und Credentials.
