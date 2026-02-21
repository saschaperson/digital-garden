---
title: AdBlock – Pi-hole & iSponsorBlockTV
publish: true
date: 2026-02-20
tags:
  - homelab
  - adblock
  - pi-hole
  - dns
---

# AdBlock – Pi-hole & iSponsorBlockTV

> Zurück zum [[Homeserver]]-Überblick.

Zwei Ebenen Werbeblocker: Pi-hole filtert DNS-Anfragen netzwerkweit, iSponsorBlockTV entfernt Sponsor-Segmente aus YouTube-Videos auf dem Fernseher.

---

## Pi-hole

### Setup

- **Image:** `jacklul/pihole:latest` (Fork mit integriertem `pihole-updatelists`)
- **Netzwerk:** `network_mode: host` – notwendig für DNS auf Port 53 und optionales DHCP
- **Config-Verzeichnis:** `./pihole/etc-pihole:/etc/pihole`

### Blocklisten

Automatisch aktualisiert über `pihole-updatelists`:

| Quelle | URL |
|--------|-----|
| Firebog Ticked Lists | `https://v.firebog.net/hosts/lists.php?type=tick` |
| mmotti Regex | `https://raw.githubusercontent.com/mmotti/pihole-regex/master/regex.list` |

### DNS-Konfiguration im Netzwerk

Damit Pi-hole wirkt, muss der Router (oder jedes Gerät einzeln) den Pi als DNS-Server verwenden:

- **Option A (empfohlen):** Im Router den DNS-Server auf die IP des Pi setzen
- **Option B:** Pi-hole als DHCP-Server nutzen (Router-DHCP deaktivieren)
- **Option C:** Pro Gerät manuell den DNS umstellen

### Web-Interface

Erreichbar unter `http://<pi-ip>/admin`. Passwort wird über die Umgebungsvariable `FTLCONF_webserver_api_password` gesetzt.

### Troubleshooting

- **DNS-Auflösung funktioniert nicht:** `nslookup google.com <pi-ip>` testen
- **Seite wird fälschlich geblockt:** Pi-hole → Query Log → Whitelist
- **Blocklisten aktualisieren sich nicht:** `docker exec pihole pihole updateGravity`
- **Container startet nicht (Port 53 belegt):** `sudo lsof -i :53` – ggf. systemd-resolved deaktivieren

---

## iSponsorBlockTV

Entfernt Sponsor-Segmente, Intros, Outros und Self-Promotion aus YouTube-Videos. Nutzt die SponsorBlock-Community-Datenbank und steuert die Wiedergabe auf Apple TV oder Smart TVs.

### Setup

- **Image:** `ghcr.io/dmunozv04/isponsorblocktv:latest`
- **Config-Verzeichnis:** `./isponsorblocktv:/app/data`
- **Netzwerk:** Standard Docker Bridge (kein Host-Modus nötig)

### Ersteinrichtung

Beim ersten Start muss das Gerät (Apple TV / Smart TV) gekoppelt werden. Die Konfiguration wird in `./isponsorblocktv/` gespeichert.

<!-- Ergänze hier die Schritte zur Kopplung mit deinem Gerät -->

### Troubleshooting

- **Segmente werden nicht übersprungen:** App auf dem TV neu starten, Container-Logs prüfen
- **Gerät nicht gefunden:** Container und TV im selben Netzwerk?
- **Nach TV-Update funktioniert es nicht mehr:** Container neu starten, ggf. neu koppeln
