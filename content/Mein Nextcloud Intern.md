---
title: Nextcloud – Intern
publish: false
date: 2026-02-20
tags:
  - homelab
  - nextcloud
  - credentials
---

# Nextcloud – Intern

> ⛔ **Nicht veröffentlichen.** Enthält Zugangsdaten und interne Pfade. Öffentliche Version: [[Nextcloud – CalDAV & CardDAV]].

---

## Zugangsdaten

| Was | Wert |
|-----|------|
| Admin-User | `sascha` |
| Admin-Passwort | <!-- hier eintragen --> |
| MySQL-User | `fiedler` |
| MySQL-Passwort | via `my_print_defaults client` auf Uberspace |
| MySQL-Datenbank | `fiedler_nextcloud` |

---

## Uberspace

- **Host:** `hippocamp.uberspace.de`
- **Account:** `fiedler`
- **SSH:** `ssh fiedler@hippocamp.uberspace.de`
- **PHP:** 8.3

### Verzeichnisstruktur

```
/var/www/virtual/fiedler/
├── html/                              → saschafiedler.com (Platzhalter)
├── nextcloud/                         → Nextcloud-Installation
└── cloud.saschafiedler.com -> nextcloud   (Symlink)

/home/fiedler/
├── nextcloud-data/                    → Datendirectory (außerhalb DocumentRoot)
├── users/                             → Mailboxen (~4.5 GB, NICHT ANFASSEN)
├── etc/php.d/                         → PHP-Konfiguration
├── bin/nextcloud-update               → Update-Script
```

### PHP-Einstellungen (`~/etc/php.d/`)

| Datei | Inhalt |
|-------|--------|
| `opcache.ini` | OPcache aktiv, 256 MB, 10k Dateien |
| `apcu.ini` | `apc.enable_cli=1` |
| `memory_limit.ini` | `memory_limit=512M` |
| `output_buffering.ini` | `output_buffering=off` |

PHP neustarten: `uberspace tools restart php`

---

## DNS (Cloudflare)

- Domain bei Namecheap, Custom DNS zeigt auf Cloudflare
- `cloud.saschafiedler.com` → CNAME → `hippocamp.uberspace.de` (Proxied)
- SSL-Modus: **Full** (nicht Flexible, nicht Full Strict)
- MX-Record: `hippocamp.uberspace.de` (DNS only – für E-Mail)

### Domains auf Uberspace

| Domain | Ziel |
|--------|------|
| `saschafiedler.com` | Platzhalter-Seite |
| `www.saschafiedler.com` | Wie oben |
| `cloud.saschafiedler.com` | Nextcloud |
| `fiedler.uber.space` | Uberspace-Standard |

---

## Nextcloud System-Config

```
trusted_domains:       cloud.saschafiedler.com
overwrite.cli.url:     https://cloud.saschafiedler.com
overwriteprotocol:     https
default_phone_region:  DE
default_locale:        en_GB
memcache.local:        \OC\Memcache\APCu
forwarded_for_headers: HTTP_CF_CONNECTING_IP
trusted_proxies:       Alle Cloudflare IPv4-Ranges (15 Einträge)
```

Vollständige Config: `php occ config:list system`

### Cronjob

```
*/5 * * * * php -f /var/www/virtual/fiedler/nextcloud/cron.php
```

---

## Wartung

### Schnell-Update

```bash
~/bin/nextcloud-update
```

### Manuelles Update

```bash
cd /var/www/virtual/fiedler/nextcloud
php updater/updater.phar -vv --no-backup --no-interaction
php occ upgrade
php occ app:update --all
```

### Status & Apps

```bash
cd /var/www/virtual/fiedler/nextcloud
php occ status
php occ app:list --enabled
php occ app:list --disabled
```

---

## Noch zu erledigen

### Kontakte von iCloud zu Nextcloud migrieren

1. Am Mac: Kontakte-App → Alle auswählen (⌘+A) → Ablage → Exportieren → vCard (.vcf)
2. In Nextcloud: Kontakte-App → Settings → „Import contacts" → .vcf hochladen
3. Geburtstagskalender sync anstoßen: `php occ dav:sync-birthday-calendar`

**Wichtig:** Nur Kontakte mit ausgefülltem Geburtstagsdatum erscheinen im Geburtstagskalender.

### Account für Freundin

```bash
cd /var/www/virtual/fiedler/nextcloud
php occ user:add freundin --display-name="Vorname"
```

Kalender teilen: Kalender-App → Drei-Punkte-Menü → Teilen → Namen eingeben.

### iOS-Geräte verbinden

1. Einstellungen → Kalender → Accounts → Account hinzufügen → Andere
2. CalDAV-Account: Server `cloud.saschafiedler.com`, User, Passwort
3. CardDAV-Account: gleich
4. Standardaccount für neue Kontakte auf Nextcloud umstellen

---

## E-Mail auf Uberspace

Mailboxen in `~/users/` (~4.5 GB). Aktiv genutzt für @saschafiedler.com. **Nicht löschen.**

---

## Bewusst nicht installiert

| Feature | Grund |
|---------|-------|
| Redis | APCu reicht für 1-2 User |
| notify_push | Polling alle 30s reicht |
| Collabora/OnlyOffice | Kein File Sharing |
| 2FA (TOTP) | Kann nachgerüstet werden |
