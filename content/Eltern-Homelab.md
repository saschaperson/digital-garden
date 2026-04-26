---
title: Eltern-Homelab
publish: false
date: 2026-04-26
tags:
  - homelab
  - eltern
  - mac-mini
---

# Eltern-Homelab

> ⛔ **Nicht veröffentlichen.** Mac mini Late 2012 als Homeserver für die Eltern. Separates Projekt — bewusst getrennt vom eigenen Homelab-Stack.

---

## Warum separat dokumentiert

Das Eltern-Homelab läuft in einem anderen LAN, mit anderer Hardware und anderen Anforderungen. Änderungen am eigenen Tailscale-Setup dürfen den Pulse Agent hier nicht brechen. Langfristig: separate Tailnets, damit Netzwerkentscheidungen unabhängig getroffen werden können.

**Empfehlung ausstehend: Tailnets trennen.** Aktuell teilen beide Setups ein Tailnet. Das schafft unnötige Abhängigkeiten. Separates Tailnet für das Eltern-Setup würde bedeuten: Pulse-Monitoring über Healthchecks.io statt direkter Tailscale-Verbindung, SSH über eigenes Tailnet der Eltern.

---

## Hardware

| Komponente | Detail |
|------------|--------|
| Modell | Mac mini Late 2012 |
| CPU | Intel Core i5-3210M (2 Kerne, 2,5 GHz) |
| RAM | 8 GB DDR3L-1600 |
| SSD | 128 GB Samsung 840 Pro — im originalen Apple-Slot |
| HDD | 500 GB Apple HDD — im iFixit Dual Drive Kit-Bracket |
| Ethernet | Gigabit (enp1s0f0) |
| WLAN | Broadcom BCM4331 — Treiber installiert, nicht konfiguriert |

### Bekannte Hardware-Punkte

**USB-Ports 3+4 defekt:** Irrelevant für headless Betrieb.

**SATA-Werte SSD — beobachten:** Scrutiny zeigt erhöhte Werte (`SATA Downshift Error Count` und `UltraDMA CRC Error Count`). Alarmiert automatisch wenn Werte steigen.

**WLAN als Fallback:** `wlp2s0` vorhanden, nicht konfiguriert. Bei Bedarf vor Ort einrichten: `sudo iwlist wlp2s0 scan`.

---

## Netzwerk

| Parameter | Wert |
|-----------|------|
| Router | eero 6 (Gateway) |
| Subnetz | 192.168.4.0/24 |
| Mac mini LAN-IP | 192.168.4.10 (feste Reservation im eero) |
| Tailscale-Hostname | mac-mini |
| Tailscale-Subnetz | 192.168.4.0/24 advertised |

---

## Betriebssystem

Debian 13 „Trixie" Minimal-Installation (Netinstall). LVM auf der SSD: `/boot/efi`, `/boot`, `lv_root` (111 GB), `lv_swap` (6 GB).

Kein UFW, kein SSH-Hardening — SSH nur über Tailscale erreichbar, kein öffentlicher Port.

---

## Tailscale

```bash
sudo tailscale up \
  --advertise-routes=192.168.4.0/24 \
  --accept-routes \
  --accept-dns=false \
  --ssh \
  --hostname=mac-mini
```

**`--accept-routes`:** Damit der Mac mini das Proxmox-Heimnetz über den dortigen Subnet-Router erreichen kann (für Pulse Agent).

**Key Expiry deaktiviert:** Remote-Server — manuelle Reauthentifizierung wäre fatal.

**GRO Forwarding:** Als systemd-Service `tailscale-gro.service` auf `enp1s0f0`.

---

## Docker

Docker CE aus dem offiziellen Docker-Repo. Log-Rotation konfiguriert (`max-size: 10m`, `max-file: 3`). Subnetz-Pool angepasst um Konflikte mit Tailscale zu vermeiden.

**Stack-Layout:** `/opt/stacks/` mit einem Verzeichnis pro Stack.

**HDD-Mount:** `/mnt/hdd` (UUID-basiert, `nofail`, `noatime`).

---

## Services

| Service | Port | Funktion |
|---------|------|----------|
| Dockge | 5001 | Docker Compose UI |
| Pi-hole | 8053 (Admin) / 53 (DNS) | DNS-Blocker, Bridge-Netzwerk |
| Jellyfin | 8096 | Media-Server, **Direct Play only** |
| Home Assistant | 8123 | Smart Home, `network_mode: host` |
| Scrutiny | 8080 | SMART-Monitoring SSD + HDD |
| Speedtest | 8082 | Stündliche Geschwindigkeitsmessung |
| Paperless-ngx | 8000 | Dokumentenmanagement für die Eltern |
| Pulse Agent | — | Monitoring via Proxmox Pulse |

### Jellyfin — Direct Play only

Intel HD 4000 (Ivy Bridge 2012) auf modernem Linux-Kernel für Transcoding unzuverlässig. Transcoding im Setup-Wizard deaktivieren. Medien in kompatiblen Formaten bereitstellen (H.264 AAC, ≤ 1080p).

### Home Assistant — geplante Integrationen

| Integration | Status |
|-------------|--------|
| EVCC | ✅ Offizielle HA-Integration |
| myVaillant | ✅ HACS: signalkraft/mypyllant |
| Aqara M100 | ✅ HomeKit-Controller oder Matter |
| Bosch Smart Home | ✅ bosch_shc |
| Fronius | ✅ Direkte LAN-Integration |

### Pi-hole — Rollout-Strategie

- Phase A: Test-Client auf Pi-hole zeigen, Query-Log beobachten
- Phase B: eero DNS → Custom → 192.168.4.10

Upstream DNS: Quad9 (9.9.9.9) + Cloudflare (1.1.1.1).

### Paperless-ngx

Admin-User: `stephan`. PostgreSQL für DB (auf SSD), Dokumente auf HDD (`/mnt/hdd/paperless/`). OCR: `deu+eng`.

---

## Monitoring

Healthchecks.io (Free Tier, Projekt „Mac mini"):

| Check | Interval | Zweck |
|-------|----------|-------|
| host-heartbeat | 5 min | Mini erreichbar |
| docker-daemon | 15 min | Docker + min. 9 Container running |
| scrutiny-smart | 24 h | SMART-Collector gelaufen |

E-Mail-Notification aktiv. Ping-URLs in `/etc/healthchecks/env`.

---

## Backup (ausstehend)

Geplant: Restic für Pi-hole Config, HA Config, Paperless Export. Jellyfin-Media kein Backup (reproduzierbar).

---

## Zugang (alle via Tailscale `mac-mini`)

| Service | URL |
|---------|-----|
| Dockge | http://mac-mini:5001 |
| Pi-hole | http://mac-mini:8053/admin |
| Jellyfin | http://mac-mini:8096 |
| Home Assistant | http://mac-mini:8123 |
| Scrutiny | http://mac-mini:8080 |
| Speedtest | http://mac-mini:8082 |
| Paperless | http://mac-mini:8000 |

Credentials in 1Password.

---

## Nächste Schritte vor Ort

1. WLAN konfigurieren (eero-SSID + Passwort via `wpa_supplicant`)
2. Mac mini an eero, einschalten
3. Via Tailscale: `ssh sascha@mac-mini` → `tailscale status`, `docker ps`
4. Jellyfin Setup-Wizard: Library anlegen, Transcoding deaktivieren
5. Home Assistant einrichten, Integrationen anlegen
6. Test-Client auf Pi-hole zeigen, Query-Log beobachten
7. Healthchecks.io → alle drei grün

**Rollback:** Tailscale-Auth in Admin-Konsole remote widerrufen → Mini ausschalten → abholen.
