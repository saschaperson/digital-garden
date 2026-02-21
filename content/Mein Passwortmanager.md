---
title: Vaultwarden
publish: true
date: 2026-02-20
tags:
  - homelab
  - self-hosting
  - security
  - passwords
---

# Vaultwarden

> Zurück zum [[Homeserver]]-Überblick. Teil von [[Homeserver#Digitale Souveränität]].

Self-hosted Bitwarden-kompatibler Passwortmanager. Leichtgewichtige Rust-Implementierung, die problemlos auf dem Raspberry Pi läuft.

---

## Setup

- **Image:** `vaultwarden/server:latest`
- **Port:** `8081:80`
- **Config-Verzeichnis:** `./vaultwarden/data:/data`
- **Zugriff:** Ausschließlich über Tailscale (`https://raspberry-pi-4.tail_____.ts.net`)
- **Registrierung:** Per Umgebungsvariable gesteuert (nach Ersteinrichtung deaktiviert)
- **Admin-Panel:** Token-geschützt über Umgebungsvariable

### Warum Tailscale-only?

Vaultwarden ist bewusst **nicht** über Cloudflare Tunnel erreichbar. Passwort-Tresore sollten nur über vertrauenswürdige Netzwerkpfade zugänglich sein. Tailscale bietet End-to-End-Verschlüsselung ohne öffentlich erreichbare Endpoints.

---

## Clients

Bitwarden-Apps und Browser-Extensions verbinden sich mit der Tailscale-URL als „Self-hosted Server":

- **iOS:** Bitwarden App → Einstellungen → Self-hosted → Server-URL eintragen
- **Browser:** Bitwarden Extension → Einstellungen → Self-hosted
- **macOS:** Bitwarden Desktop App → gleiche Einstellung

### Wichtig

Vaultwarden ist nur erreichbar, wenn Tailscale auf dem jeweiligen Gerät aktiv ist. Offline-Zugriff funktioniert trotzdem – Bitwarden cached den Tresor lokal.

---

## Backup

<!-- Ergänze hier deine Backup-Strategie -->

Die SQLite-Datenbank liegt in `./vaultwarden/data/`. Ein regelmäßiges Backup dieses Verzeichnisses sichert alle Passwörter und Anhänge.

---

## Troubleshooting

- **Bitwarden-App kann sich nicht verbinden:** Tailscale aktiv? URL korrekt mit `https://`?
- **Admin-Panel nicht erreichbar:** `/admin` an die URL anhängen, Token prüfen
- **Registrierung schlägt fehl:** `SIGNUPS_ALLOWED` in der `.env` prüfen
- **Nach Update kein Login möglich:** `docker logs vaultwarden` – ggf. DB-Migration nötig
