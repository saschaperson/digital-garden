---
title: Homelab – Privat
publish: false
date: 2026-04-26
tags:
  - homelab
  - privat
  - credentials
---

# Homelab – Privat

> ⛔ **Nicht veröffentlichen.** Enthält alle Identifier, Credentials und sensiblen Konfigurationsdetails. Öffentliche Dokumentation: [[Homelab]].

---

## Netzwerk

### Geräte-IPs

| Gerät | IP |
|-------|----|
| Fritz!Box | 192.168.178.1 |
| Powerline | 192.168.178.2 |
| OptiPlex (Proxmox) | 192.168.178.3 |
| Raspberry Pi 4 (HA) | 192.168.178.4 |
| CT 101 infrastructure | 192.168.178.10 |
| CT 102 servarr | 192.168.178.11 |
| CT 103 media | 192.168.178.12 |
| CT 104 services | 192.168.178.13 |
| CT 105 photos | 192.168.178.14 |
| CT 106 documents | 192.168.178.15 |
| Apple TV | 192.168.178.20 |
| Philips TV | 192.168.178.21 |
| PlayStation | 192.168.178.22 |
| Roborock | 192.168.178.30 |
| Bambu A1 | 192.168.178.31 |
| ESP32-S3 | 192.168.178.32 |
| Aqara Hub | 192.168.178.33 |

### Port-Übersicht

| CT | Service | Port | Extern |
|----|---------|------|--------|
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
| 104 | Actual Budget | 5006 | Tailscale Serve |
| 104 | Speedtest Tracker | 8765 | Nein |
| 104 | Homarr | 7575 | Tunnel |
| 104 | BetterBahn | 3000 | Tunnel (Access) |
| 104 | Homebox | 3100 | Nein |
| 104 | Bambuddy | 8000 | Nein |
| 105 | Immich | 2283 | Tailscale |
| 106 | Paperless-ngx | 8000 | Tailscale |

---

## Tailscale

| Eigenschaft | Wert |
|-------------|------|
| Tailnet | tail4a9632.ts.net |
| Proxmox Hostname | proxmox.tail4a9632.ts.net |
| Actual Budget URL | https://proxmox.tail4a9632.ts.net:5006 |
| Key Expiry | Deaktiviert |

---

## Cloudflare

| Eigenschaft | Wert |
|-------------|------|
| Domain | saschafiedler.com |
| SSL-Modus Nextcloud | Full |

### Tunnel-Subdomains

| Subdomain | Service | Schutz |
|-----------|---------|--------|
| jellyfin.saschafiedler.com | Jellyfin (CT 103) | Eigener Login |
| seerr.saschafiedler.com | Seerr (CT 102) | Eigener Login |
| home.saschafiedler.com | Homarr (CT 104) | Eigener Login |
| betterbahn.saschafiedler.com | BetterBahn (CT 104) | Cloudflare Access (E-Mail-OTP) |

**Cloudflare Access Cookie-Domain:** `.saschafiedler.com` — ein OTP-Login gilt für alle Access-geschützten Subdomains.

### DNS-Records

| Record | Typ | Wert | Proxy |
|--------|-----|------|-------|
| cloud.saschafiedler.com | CNAME | hippocamp.uberspace.de | Proxied |
| saschafiedler.com | MX | hippocamp.uberspace.de | DNS only |

---

## Proxmox API-Tokens

| Token | Rolle | Verwendung |
|-------|-------|------------|
| pulse@pam!monitoring | PVEAuditor | Pulse Monitoring |
| homarr@pam!monitoring | PVEAuditor | Homarr Proxmox-Widget |

---

## Storage — Drives

### Zuordnung

| Dev | Seriennummer | Modell | Gehäuse | Mount | Pool |
|-----|-------------|--------|---------|-------|------|
| sda | S1DHNEADA06832H | Samsung SSD 840 EVO 500GB | intern | — | Boot (Proxmox) |
| sdb | WD-WX52D6307883 | WDC WD50NPJZ-00CBYT0 | Intenso USB | /mnt/disk2 | MergerFS |
| sdc | WCJ7X3V8 | Seagate ST5000LM000-2AN170 | Seagate Portable (0bc2:2344) | /mnt/disk1 | MergerFS |
| sdd | WCJB5VXG | Seagate ST5000LM000-2U8170 | Seagate Expansion SW (0bc2:203a) | /mnt/disk3 | MergerFS ⚠️ |
| (sde) | WCJ0G4GW | Seagate ST5000LM000-2AN170 | WD MyPassport (USB) | — | SnapRAID Parity (geplant) |

5. Disk noch nicht angeschlossen — wird nach SATA-Migration als SnapRAID-Parity-Disk eingebunden. Dev-Name sde vorläufig, hängt von Anschlussreihenfolge ab.

### SMART-Baseline (Stand 2026-04-26)

| Drive | Serial | POH | Reallocated (ID5) | Pending (ID197) | Uncorrectable (ID198) | CRC-Fehler (ID199) | Befund |
|-------|--------|-----|------------------|-----------------|----------------------|-------------------|--------|
| sda (Samsung 840 EVO) | S1DHNEADA06832H | 2 Jahre | 0 | — | 0 | 0 | ✅ Sauber |
| sdb (Intenso/WD) | WD-WX52D6307883 | 44 Tage | 0 | 0 | 0 | 0 | ✅ Sauber |
| sdc (Seagate Portable) | WCJ7X3V8 | 1.291h | 0 | 0 | 0 | 0 | ✅ Sauber |
| sdd (Seagate Expansion) | WCJB5VXG | 6.195h | 0 | 0 | 0 | **16.332** | ⚠️ CRC-Fehler |
| (sde) Parity (WD MyPassport) | WCJ0G4GW | 2.790h | 0 | 0 | 0 | 0 | ✅ Sauber |

**SSD-spezifische Werte (sda Samsung 840 EVO):**

| Attribut | Wert | Bedeutung |
|----------|------|-----------|
| Wear Range Delta (ID177) | 82 | Verschleißspanne zwischen am meisten und am wenigsten genutzten Blöcken — unter 100 okay |
| Good Block Count (ID235) | 99 | Verfügbare reservierte Blöcke — sinkt mit Verschleiß |
| Total LBAs Written (ID241) | 99 | Gesamtschreiblast — noch weit vom Limit |
| Temperatur | 57°C | Oberes Ende des akzeptablen Bereichs (< 60°C). Beobachten. |

**sdd CRC-Verlauf:**

| Datum | CRC-Fehler (ID199) |
|-------|--------------------|
| 2026-04-25 | 16.332 |
| (nächste Messung) | |

CRC-Fehler sind fast immer Verbindungsfehler (Kabel, USB-Gehäuse-Bridge) — nicht unbedingt die Disk. USB-Kabel tauschen vor nächstem Dauerbetrieb. Wenn der Wert nach Wiedereinstecken weiter steigt, ist das Gehäuse defekt.

### Scrutiny

Läuft auf dem Proxmox-Host direkt, Port 8080. Nur sda und sdb werden überwacht (sdc/sdd: USB-Bridge blockiert SMART-Passthrough). Nach SATA-Migration: alle vier Drives in Scrutiny sichtbar.

| Eigenschaft | Wert |
|-------------|------|
| URL | http://192.168.178.3:8080 |
| Compose | /opt/scrutiny/ |
| Collector Config | /opt/scrutiny/config/collector.yaml |

---

## Storage — Disk-UUIDs

| Disk | UUID | Mount |
|------|------|-------|
| sdc1 | f1e84637-fdbd-423b-834e-730ef00c2330 | /mnt/disk1 |
| sdb1 | a15e7212-8b53-433e-8a1d-bbd595640f68 | /mnt/disk2 |
| sdd1 | 54312f02-42cc-468b-9578-696c0613f051 | /mnt/disk3 |

---

## Nextcloud (Uberspace)

| Eigenschaft | Wert |
|-------------|------|
| Host | hippocamp.uberspace.de |
| Account | fiedler |
| SSH | `ssh fiedler@hippocamp.uberspace.de` |
| Admin-User | sascha |
| MySQL-DB | fiedler_nextcloud |
| Nextcloud-Pfad | /var/www/virtual/fiedler/nextcloud/ |
| Daten-Pfad | /home/fiedler/nextcloud-data/ |

```bash
# Cronjob
*/5 * * * * php -f /var/www/virtual/fiedler/nextcloud/cron.php

# PHP neu starten
uberspace tools restart php

# Update-Script
~/bin/nextcloud-update
```

---

## Docker .env Template

```bash
TZ=Europe/Berlin
PUID=1000
PGID=1000

# Pi-hole
PIHOLE_FTL_API_PASSWORD=

# Cloudflare Tunnel
CLOUDFLARED_TUNNEL_TOKEN=

# Vaultwarden
VW_DOMAIN=
VW_SIGNUPS_ALLOWED=false
VW_ADMIN_TOKEN=
```

---

## Geplante Hardware-Upgrades

| Komponente | Zweck | Preis |
|------------|-------|-------|
| ASM1166 M.2-SATA-Controller (6 Port) | USB → SATA für HDDs | 10–18 € |
| 4. 5 TB HDD | SnapRAID Parity | 80–120 € |
| 5V/12V SATA-Netzteil + Splitter | Stromversorgung HDDs | 10–15 € |
| 128 GB SATA 2,5" SSD (falls nötig) | Boot-SSD nach M.2-Slot-Umbau | 15–18 € |
| RAM Upgrade auf 32 GB | Mehr Headroom für Container | ~40–60 € |

### Migration USB → SATA (wenn Controller da ist)

1. Boot-SSD auf 2,5" SATA umziehen, M.2-Slot freimachen
2. ASM1166-Controller in M.2-Slot einsetzen
3. HDDs per SATA-Kabel anschließen
4. fstab: UUIDs bleiben gleich — nur Transport ändert sich, kein Datenverlust
5. MergerFS-Pool funktioniert sofort weiter
6. Vierte HDD anschließen, formatieren, SnapRAID konfigurieren

**Hinweis:** ASM1166 bevorzugt gegenüber JMB585 — JMB585 droppt im Idle gelegentlich Disks. Bei paralleler Stromversorgung auf ausreichend starkes Netzteil achten (4× 2,5" HDDs an einem Y-Kabel ist typische Instabilitätsquelle).

---

## Samba-Share (HA-Backup)

| Einstellung | Wert |
|-------------|------|
| Share-Name | homeassistant_backup (Unterstrich, kein Bindestrich) |
| Host | 192.168.178.3 |
| Port | 445 |
