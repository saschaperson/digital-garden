---
title: Nextcloud – CalDAV & CardDAV
publish: true
date: 2026-02-20
tags:
  - homelab
  - self-hosting
  - nextcloud
  - caldav
---

# Nextcloud – CalDAV & CardDAV

> Zurück zum [[Homeserver]]-Überblick. Teil von [[Homeserver#Digitale Souveränität]].

Schlanke Nextcloud-Instanz, die ausschließlich als CalDAV/CardDAV-Server für Kontakte und Kalender dient. Kein File Sharing, kein Office, kein Talk. Hauptzweck: automatischer Geburtstagskalender, synchronisiert auf iOS und das [[Home Assistant]] iPad Dashboard.

---

## Infrastruktur

- **Hosting:** Uberspace (Shared Hosting, kein eigener Server)
- **Nextcloud-Version:** 32.x
- **PHP:** 8.3
- **Datenbank:** MySQL
- **Domain:** Per Cloudflare-DNS (CNAME, Proxy-Modus)
- **SSL:** Cloudflare SSL-Modus **Full** (nicht Flexible, nicht Full Strict)

### Warum Uberspace statt Pi?

Nextcloud braucht einen öffentlich erreichbaren Webserver für CalDAV-Sync auf iOS-Geräten (unterwegs, ohne VPN). Uberspace bietet das ohne eigene Infrastruktur. Der Pi ist für lokale Services besser geeignet.

---

## Aktive Apps (nur das Nötigste)

- **calendar** – Kalender-UI und CalDAV-Backend
- **contacts** – Kontakte-UI und CardDAV-Backend
- **dav** – CalDAV/CardDAV Core
- **bruteforcesettings** – Login-Schutz
- **password_policy** – Passwort-Richtlinien

Alles andere (Dashboard, Files Sharing, Activity, Photos, Federation, etc.) ist bewusst deaktiviert.

---

## Geburtstagskalender

Nextcloud generiert automatisch einen Kalender „Contact birthdays" aus allen Kontakten, die ein Geburtstagsdatum haben. Dieser Kalender wird per CalDAV synchronisiert auf:

1. **iOS-Geräte** (iPhone, iPad) über die native CalDAV-Integration
2. **Home Assistant** über die CalDAV-Integration in `configuration.yaml`

### Sync manuell anstoßen

```bash
cd /var/www/virtual/<user>/nextcloud
php occ dav:sync-birthday-calendar
```

---

## iOS-Einrichtung

Auf dem iPhone/iPad:

1. Einstellungen → Kalender → Accounts → Account hinzufügen → **Andere**
2. **CalDAV-Account hinzufügen** – Server, Benutzername, Passwort eingeben
3. Dasselbe für **CardDAV-Account**
4. Standard-Account für neue Kontakte auf Nextcloud umstellen

---

## Wartung

### Update

```bash
cd /var/www/virtual/<user>/nextcloud
php updater/updater.phar -vv --no-backup --no-interaction
php occ upgrade
php occ app:update --all
```

### Status prüfen

```bash
php occ status
php occ app:list --enabled
```

### Cronjob

Alle 5 Minuten läuft `cron.php` für Hintergrundaufgaben. Modus: `cron` (nicht AJAX).

---

## Troubleshooting

- **Redirect-Loop nach Login:** Cloudflare SSL-Modus auf „Full" prüfen
- **„Access through untrusted domain":** Trusted Domains in der Nextcloud-Config prüfen
- **Geburtstagskalender leer:** Kontakte brauchen ein ausgefülltes Geburtstagsdatum, `dav:sync-birthday-calendar` manuell ausführen
- **Nextcloud langsam:** APCu als Memcache konfiguriert? Cronjob aktiv?
- **HA Calendar Entity fehlt:** CalDAV-URL prüfen, HA nach Config-Änderung neu starten
