---
title: Nextcloud
publish: true
date: 2026-04-26
tags:
  - homelab
  - self-hosting
  - nextcloud
  - caldav
---

# Nextcloud

> Schlanke Nextcloud-Instanz als CalDAV/CardDAV-Server. Läuft extern auf Uberspace (Shared Hosting), nicht auf dem Homelab-Stack. Zurück zum [[Homelab]]-Überblick.

---

## Warum extern und nicht selbst gehostet

CalDAV-Sync auf iOS-Geräten benötigt einen öffentlich erreichbaren Server — auch unterwegs, ohne VPN. Der Pi 4 und der OptiPlex sind primär für lokale Services ausgelegt. Uberspace bietet das ohne eigene Infrastruktur, ohne Zertifikats-Management und ohne Abhängigkeit vom Heimanschluss.

Solange Nextcloud ausschließlich für Kalender und Kontakte genutzt wird (kein File Sharing, kein Office), ist Shared Hosting die wartungsärmere Lösung.

---

## Infrastruktur

| Eigenschaft | Wert |
|-------------|------|
| Hosting | Uberspace (Shared Hosting) |
| PHP | 8.3 |
| Datenbank | MySQL |
| Caching | APCu (reicht für 1–2 User) |
| SSL | Cloudflare SSL-Modus Full |

### DNS

Domain bei Namecheap, Custom DNS auf Cloudflare. `cloud.<deine-domain>` → CNAME → Uberspace-Host (Proxied). SSL-Modus **Full** — nicht Flexible (unsicher) und nicht Full Strict (Zertifikat-Mismatch mit Uberspace).

---

## Aktive Apps

Nur das absolut Nötigste:

- **calendar** — Kalender-UI und CalDAV-Backend
- **contacts** — Kontakte-UI und CardDAV-Backend
- **dav** — CalDAV/CardDAV Core
- **bruteforcesettings** — Login-Schutz
- **password_policy** — Passwort-Richtlinien

Alles andere (Dashboard, Files Sharing, Activity, Photos, Federation) ist bewusst deaktiviert.

---

## Geburtstagskalender

Nextcloud generiert automatisch „Contact birthdays" aus allen Kontakten mit Geburtstagsdatum. Dieser Kalender wird synchronisiert auf iOS-Geräte und [[Home Assistant]] (iPad Dashboard).

Nur Kontakte mit ausgefülltem Geburtstagsdatum erscheinen im Kalender.

```bash
# Sync manuell anstoßen
php occ dav:sync-birthday-calendar
```

---

## iOS-Einrichtung

1. Einstellungen → Kalender → Accounts → Account hinzufügen → **Andere**
2. CalDAV-Account: Server, Benutzername, Passwort
3. Dasselbe für CardDAV-Account
4. Standard-Account für neue Kontakte auf Nextcloud umstellen

---

## Wartung

```bash
# Update
php updater/updater.phar -vv --no-backup --no-interaction
php occ upgrade
php occ app:update --all

# Status
php occ status
php occ app:list --enabled
```

Cronjob läuft alle 5 Minuten (`cron.php`). Modus: `cron` (nicht AJAX).

---

## System-Konfiguration

```
trusted_domains:       cloud.<deine-domain>
overwrite.cli.url:     https://cloud.<deine-domain>
overwriteprotocol:     https
default_phone_region:  DE
default_locale:        en_GB
memcache.local:        \OC\Memcache\APCu
forwarded_for_headers: HTTP_CF_CONNECTING_IP
trusted_proxies:       Cloudflare IPv4-Ranges
```

Credentials und Pfade in `[[Homelab – Privat]]`.

---

## Troubleshooting

- **Redirect-Loop nach Login:** Cloudflare SSL-Modus auf „Full" prüfen (nicht Flexible)
- **„Access through untrusted domain":** Trusted Domains in der Config prüfen
- **Geburtstagskalender leer:** Kontakte brauchen Geburtstagsdatum, `dav:sync-birthday-calendar` manuell ausführen
- **HA Calendar Entity fehlt:** CalDAV-URL prüfen, HA nach Config-Änderung neu starten
