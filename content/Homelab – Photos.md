# Homelab – Photos

> CT 105 — Immich als Google-Photos-Alternative. Gesichtserkennung und Smart Search über OpenVINO auf der iGPU. Zurück zum [[Homelab]]-Überblick.

---

## Container

| Eigenschaft | Wert |
| --- | --- |
| CT ID | 105 |
| Hostname | `photos` |
| IP | 192.168.178.14 |
| Cores | 4 |
| RAM | 4 GB |
| Swap | 1 GB |
| Disk | 16 GB (local-zfs) |
| iGPU | ✅ `dev0: /dev/dri/renderD128,mode=0666` |
| Bind-Mount | `/mnt/storage/Photos` → `/mnt/photos` (rw) |
| Start Order | 5 |

4 Cores und 4 GB RAM, weil Immich bei ML-Jobs (Gesichtserkennung, Smart Search, OCR) deutlich mehr zieht als andere Services. Langfristig auf 6–8 GB erhöhen nach dem Host-Upgrade auf 32 GB.

## Services

Immich besteht aus vier Containern:

| Service | Funktion |
| --- | --- |
| immich-server | API, Web-UI, Video-Transcoding (VAAPI) |
| immich-machine-learning | Gesichtserkennung, CLIP, OCR (OpenVINO auf iGPU) |
| immich-postgres | PostgreSQL mit VectorChord (Datenbank auf SSD!) |
| immich-redis | Valkey 9 — Cache und Job Queue |

### Hardware-Beschleunigung

**Video-Transcoding:** VAAPI. Konfiguriert in Immich Admin → Video Transcoding → Hardware Acceleration = VAAPI, Hardware Decoding aktiviert.

**Machine Learning:** OpenVINO. Das ML-Image `release-openvino` nutzt die iGPU für Gesichtserkennung (buffalo_l), Smart Search (ViT-B-32) und OCR (PP-OCRv5). Verifiziert: `OpenVINOExecutionProvider` wird vor `CPUExecutionProvider` geladen. Die iGPU wird mit CT 103 (Jellyfin) geteilt.

## Storage

```
CT 105 Root-Disk (16 GB, ZFS SSD)
├── /opt/immich/              Compose, .env, hwaccel-Files
└── /opt/immich/postgres/     PostgreSQL-Daten (MUSS auf SSD)

/mnt/photos (Bind-Mount → /mnt/storage/Photos)
├── library/                  Hochgeladene Originale
├── thumbs/                   Generierte Thumbnails
├── encoded-video/            Transkodierte Videos
├── upload/                   Temporärer Upload
└── backups/                  Automatische DB-Dumps
```

PostgreSQL auf der SSD, Fotos auf dem HDD-Pool. Immich ist da strikt: DB-Performance bricht auf HDDs ein.

**Berechtigungen:** `chown -R 100000:100000 /mnt/storage/Photos` auf dem Host. Immich läuft als root im Container (UID 0 = Host-UID 100000).

## Compose

```yaml
# /opt/immich/docker-compose.yml
# Basis: offizielle Compose von https://github.com/immich-app/immich/releases/latest
# Anpassungen: VAAPI Transcoding + OpenVINO ML aktiviert

name: immich

services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    extends:
      file: hwaccel.transcoding.yml
      service: vaapi
    volumes:
      - ${UPLOAD_LOCATION}:/data
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    ports:
      - '2283:2283'
    depends_on:
      - redis
      - database
    restart: always
    healthcheck:
      disable: false

  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}-openvino
    extends:
      file: hwaccel.ml.yml
      service: openvino
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    restart: always
    healthcheck:
      disable: false

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:9
    healthcheck:
      test: redis-cli ping || exit 1
    restart: always

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    shm_size: 128mb
    restart: always
    healthcheck:
      disable: false

volumes:
  model-cache:
```

Die `hwaccel.transcoding.yml` und `hwaccel.ml.yml` werden von Immich bereitgestellt und liegen im selben Verzeichnis.

### .env (Template)

```bash
UPLOAD_LOCATION=/mnt/photos
DB_DATA_LOCATION=./postgres
TZ=Europe/Berlin
IMMICH_VERSION=v2
DB_PASSWORD=<REDACTED>
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
MACHINE_LEARNING_WORKER_TIMEOUT=300
```

## Externer Zugriff

Kein Cloudflare Tunnel. Die Immich Mobile App kommuniziert per API und funktioniert nicht hinter Cloudflare Access (gleiches Problem wie Infuse bei Jellyfin). Zugriff von unterwegs über Tailscale.

## Tipps

- **Große Imports:** ML-Concurrency in Admin → Job Queues auf 1 lassen bei 4 GB RAM. Nachts laufen lassen.
- **Backup:** Immich erstellt automatisch DB-Dumps unter `/mnt/photos/backups/`. Container-Configs werden wöchentlich durch das Host-Backup-Script gesichert.
- **Updates:** `IMMICH_VERSION=v2` pinnt auf die aktuelle Major-Version. Vor Updates immer Release Notes lesen.
