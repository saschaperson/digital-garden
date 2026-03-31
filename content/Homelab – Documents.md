# Homelab – Documents

> CT 106 — Paperless-ngx als digitales Dokumentenarchiv. OCR in Deutsch und Englisch. Zurück zum [[Homelab]]-Überblick.

---

## Container

| Eigenschaft | Wert |
| --- | --- |
| CT ID | 106 |
| Hostname | `documents` |
| IP | 192.168.178.15 |
| Cores | 2 |
| RAM | 2 GB |
| Swap | 512 MB |
| Disk | 12 GB (local-zfs) |
| Bind-Mount | `/mnt/storage/Documents` → `/mnt/documents` (rw) |
| Start Order | 6 |

## Services

| Service | Funktion |
| --- | --- |
| Paperless-ngx | Web-UI, Dokumentenverarbeitung, OCR |
| Redis | Task Queue für Celery Worker |
| Gotenberg | Office-Dokument-Konvertierung (Word, Excel, PowerPoint) |
| Tika | Metadata-Extraktion für erweiterte Dateitypen |

### Design-Entscheidungen

**SQLite statt PostgreSQL:** Spart einen Container und ~500 MB RAM. Für einen Einzelbenutzer mit ein paar hundert Dokumenten pro Jahr ausreichend. Migration auf PostgreSQL jederzeit möglich.

**Tika + Gotenberg:** Ohne die beiden kann Paperless nur PDFs und Bilder verarbeiten. Mit ihnen kommen Word, Excel, PowerPoint, E-Mails und weitere Formate dazu. Braucht etwas mehr RAM, lohnt sich aber.

**OCR-Sprachen:** `deu+eng` — Deutsch und Englisch.

## Storage

```
CT 106 Root-Disk (12 GB, ZFS SSD)
├── /opt/paperless/           Compose-File
├── /opt/paperless/data/      SQLite DB, Search Index, Classifier
├── /opt/paperless/consume/   Hier Dokumente reinwerfen → auto-Processing
└── /opt/paperless/export/    Für manuelle Exports/Backups

/mnt/documents (Bind-Mount → /mnt/storage/Documents)
└── documents/
    ├── originals/            Originaldateien
    └── archive/              OCR-verarbeitete PDFs
```

**Berechtigungen:** `chown -R 101000:101000 /mnt/storage/Documents` auf dem Host. Paperless läuft als User 1000 im Container (UID 1000 = Host-UID 101000).

## Compose

```yaml
# /opt/paperless/docker-compose.yml
services:
  broker:
    image: docker.io/library/redis:7
    container_name: paperless-redis
    restart: unless-stopped
    volumes:
      - redisdata:/data

  webserver:
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    container_name: paperless
    restart: unless-stopped
    depends_on:
      - broker
      - gotenberg
      - tika
    ports:
      - "8000:8000"
    volumes:
      - ./data:/usr/src/paperless/data
      - /mnt/documents:/usr/src/paperless/media
      - ./export:/usr/src/paperless/export
      - ./consume:/usr/src/paperless/consume
    environment:
      PAPERLESS_REDIS: redis://broker:6379
      PAPERLESS_TIKA_ENABLED: 1
      PAPERLESS_TIKA_GOTENBERG_ENDPOINT: http://gotenberg:3000
      PAPERLESS_TIKA_ENDPOINT: http://tika:9998
      PAPERLESS_TIME_ZONE: Europe/Berlin
      PAPERLESS_OCR_LANGUAGE: deu+eng
      PAPERLESS_CONSUMER_POLLING: 30
      PAPERLESS_WEBSERVER_WORKERS: 1
      USERMAP_UID: 1000
      USERMAP_GID: 1000

  gotenberg:
    image: docker.io/gotenberg/gotenberg:8
    container_name: paperless-gotenberg
    restart: unless-stopped
    command:
      - "gotenberg"
      - "--chromium-disable-javascript=true"
      - "--chromium-allow-list=file:///tmp/.*"

  tika:
    image: docker.io/apache/tika:latest
    container_name: paperless-tika
    restart: unless-stopped

volumes:
  redisdata:
```

`PAPERLESS_WEBSERVER_WORKERS: 1` spart RAM. Für einen Einzelbenutzer reicht ein Worker.

## Dokumente einbringen

Drei Wege:

1. **Consume-Ordner:** Datei nach `/opt/paperless/consume/` kopieren → automatische Verarbeitung (Polling alle 30 Sekunden).
2. **Web-Upload:** Über die Web-UI → Drag & Drop.
3. **E-Mail-Import:** Unter Settings → Mail ein IMAP-Postfach konfigurieren. Rechnungen an eine dedizierte Adresse weiterleiten.

Kein dedizierter Scanner nötig. Die iOS Dateien-App hat einen eingebauten Dokumentenscanner. PDF per Share-Sheet oder in den Consume-Ordner.

## Externer Zugriff

Nur LAN und Tailscale. Kein Cloudflare Tunnel — Paperless ist ein Einzelbenutzer-Service.
