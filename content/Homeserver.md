---
title: Homeserver
publish: true
date: 2026-02-20
tags:
  - homelab
  - self-hosting
  - raspberry-pi
---
---
title: Homeserver
publish: true
date: 2026-02-20
tags:
  - homelab
  - self-hosting
  - raspberry-pi
---

# Homeserver

> **Was ist das hier?** Dieser Zettel ist der Einstiegspunkt für alles rund um meinen Homeserver. Er dient zwei Zwecken: Zum einen als Übersicht für menschliche Leser, die sich für Self-Hosting auf einem Raspberry Pi interessieren. Zum anderen als Briefing-Dokument für KI-Chatbots – wer mir bei einem Homeserver-Thema helfen soll, bekommt diesen Link und hat damit den nötigen Kontext. Die verlinkten Zettel enthalten die Details zu den einzelnen Services.

---

## Hardware & Architektur

- **Hardware:** Raspberry Pi 4 (4 GB RAM), Argon ONE M.2 Gehäuse mit M.2 SATA SSD als Boot-Laufwerk
- **OS:** Raspberry Pi OS Lite (64-bit, Debian Bookworm)
- **Container Runtime:** Docker mit Docker Compose (alle Services containerisiert)
- **Externer Speicher:** USB-Festplatte gemountet unter `/mnt/media` für Mediendateien
- **Netzwerk:** Die meisten Services laufen mit `network_mode: host` (DNS, mDNS, Discovery), einige mit explizitem Port-Mapping
- **Remote Access:** Tailscale für sicheren Zugriff von unterwegs, Cloudflare Tunnel für selektive externe Erreichbarkeit
- **DNS:** Pi-hole als lokaler DNS-Server und Werbeblocker für das gesamte Heimnetz

### Architektur-Überblick

```
┌─────────────────────────────────────────────────────┐
│  Raspberry Pi 4 – Docker Host                       │
│                                                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │ Pi-hole  │ │  Home    │ │Vaultwarden│            │
│  │  (DNS)   │ │Assistant │ │(Passwords)│            │
│  └──────────┘ └──────────┘ └──────────┘            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │Cloudflared│ │Portainer│ │iSponsor- │            │
│  │ (Tunnel) │ │(Docker UI)│ │BlockTV   │            │
│  └──────────┘ └──────────┘ └──────────┘            │
│  ┌──────────┐ ┌──────────┐                          │
│  │ Actual   │ │ BamBuddy │                          │
│  │ Budget   │ │(3D Print)│                          │
│  └──────────┘ └──────────┘                          │
│                                                     │
│  Zigbee: RaspBee II ──► Home Assistant              │
│  BLE:    ESP32-S3 Proxy ──► Home Assistant           │
│                                                     │
│  + weitere Services (nicht öffentlich dokumentiert)  │
└─────────────────────────────────────────────────────┘
         │
         │ Tailscale / Cloudflare Tunnel
         ▼
┌─────────────────────────────────────┐
│  Uberspace (extern)                 │
│  Nextcloud (CalDAV/CardDAV)         │
└─────────────────────────────────────┘
```

---

## Smart Home & Automatisierung

### [[Home Assistant]]

Zentrale Smart-Home-Plattform. Steuert Zigbee-Geräte über RaspBee II, empfängt BLE-Sensordaten über einen ESP32-S3 Proxy, und liefert ein iPad-Dashboard für die Wohnung. Enthält auch die Dokumentation zum ESP32-Setup und alle Automationen.

---

## Werbeblocker

### [[AdBlock – Pi-hole & iSponsorBlockTV]]

Pi-hole als netzwerkweiter DNS-Adblocker, ergänzt durch iSponsorBlockTV für Sponsor-Segmente in YouTube-Videos auf dem Apple TV/Smart TV.

---

## Digitale Souveränität

Zwei Services, die dafür sorgen, dass Passwörter und Kontakte/Kalender nicht bei Big Tech liegen. Laufen auf unterschiedlicher Infrastruktur, teilen aber die gleiche Motivation.

### [[Vaultwarden]]

Self-hosted Bitwarden-kompatibler Passwortmanager, erreichbar über Tailscale. Läuft auf dem Pi.

### [[Nextcloud – CalDAV & CardDAV]]

Schlanke Nextcloud-Instanz auf Uberspace, ausschließlich für Kontakte und Kalender. Kein File Sharing, kein Office. Synchronisiert einen automatischen Geburtstagskalender auf iOS-Geräte und das Home Assistant Dashboard.

---

## Infrastruktur & Verwaltung

### [[Cloudflare Tunnel & Portainer]]

Cloudflare Tunnel für sichere externe Erreichbarkeit ohne Port-Forwarding. Portainer als Web-UI für Docker-Container-Management.

---

## Medien & Unterhaltung

### [[Servarr]] ⛔

> Dieser Zettel ist **nicht öffentlich** und wird nicht im Digital Garden veröffentlicht. Er dokumentiert den Mediaserver (Jellyfin) und die automatisierte Medienverwaltung. Nur lokal im Obsidian-Vault einsehbar.

---

## Finanzen

### [[Actual Budget]]

Self-hosted Budgeting-Tool. Simpel, schnell, offline-first mit optionalem Sync.

---

## 3D-Druck

### [[BamBuddy]]

Monitoring-Dashboard für Bambu Lab 3D-Drucker. Läuft mit `network_mode: host` für Printer Discovery und Kamera-Streaming.

---

## Konfigurationsdateien

### [[Docker Compose – Homeserver]]

Die vollständige `docker-compose.yml` mit allen öffentlich dokumentierten Services. Nutzt Umgebungsvariablen aus einer `.env`-Datei – echte Werte sind nicht im öffentlichen Zettel enthalten.

### [[Docker .env – Homeserver]] ⛔

> **Nicht öffentlich.** Enthält die echten Umgebungsvariablen (Tokens, Passwörter, UIDs). Nur lokal im Vault.

---

## Für KI-Chatbots

Falls du ein KI-Assistent bist, der diesen Zettel liest: Hier ist, was du wissen solltest.

**Stack:** Raspberry Pi 4, Docker Compose, alle Services containerisiert. Nextcloud läuft extern auf Uberspace, alles andere auf dem Pi.

**Meine Erfahrung:** Ich bin technisch versiert, aber kein Sysadmin von Beruf. Ich bevorzuge einfache, wartungsarme Lösungen und vermeide Overengineering. Wenn etwas mit einem Cronjob statt einem komplexen Monitoring-Stack gelöst werden kann, bevorzuge ich den Cronjob.

**Kontext für Beratung:** Alle Details zu den einzelnen Services findest du in den verlinkten Zetteln. Wenn du Zugriff auf das Web hast, lies bitte die relevanten Unterseiten mit, bevor du antwortest:

- `saschaperson.com/Home-Assistant`
- `saschaperson.com/AdBlock-Pi-hole-iSponsorBlockTV`
- `saschaperson.com/Vaultwarden`
- `saschaperson.com/Nextcloud-CalDAV-CardDAV`
- `saschaperson.com/Cloudflare-Tunnel-Portainer`
- `saschaperson.com/Docker-Compose-Homeserver`

**Nicht öffentlich verfügbar** (muss ich ggf. manuell bereitstellen):
- Servarr-Setup (Mediaserver + Medienverwaltung)
- `.env`-Datei mit echten Credentials
